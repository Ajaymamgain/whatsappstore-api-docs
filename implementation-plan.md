# WhatsApp Business API Integration Implementation Plan

## Introduction

This document outlines the strategic plan for integrating the WhatsApp Business API into the WhatsAppStore application. It provides a phased approach to implementation, focusing on key functionalities that will enhance customer engagement and drive sales through WhatsApp.

## Prerequisites

Before beginning implementation, ensure you have:

1. **WhatsApp Business Account**: Apply through Meta for Business
2. **WhatsApp Business API Access**: Complete the verification process
3. **Meta Developer Account**: For creating apps and accessing the API
4. **Phone Number**: A dedicated phone number for WhatsApp Business
5. **SSL Secured Backend**: For receiving webhooks securely

## Phase 1: Authentication & Basic Configuration (Week 1)

### 1.1 Set Up Meta Developer App

- Create a new app in Meta Developer Portal
- Configure WhatsApp integration
- Generate and store API credentials securely
- Set permission scopes for WhatsApp Business API

### 1.2 Authentication Service Implementation

```typescript
// lib/whatsapp/auth-service.ts
import axios from 'axios';

class WhatsAppAuthService {
  private clientId: string;
  private clientSecret: string;
  private tokenStorage: any; // Use a proper storage solution in production
  
  constructor(clientId: string, clientSecret: string, tokenStorage: any) {
    this.clientId = clientId;
    this.clientSecret = clientSecret;
    this.tokenStorage = tokenStorage;
  }
  
  async getAccessToken(): Promise<string> {
    // Check if we have a valid cached token
    const cachedToken = await this.tokenStorage.getToken();
    if (cachedToken && cachedToken.expiresAt > Date.now()) {
      return cachedToken.token;
    }
    
    // Otherwise, get a new token
    const response = await axios.post('https://graph.facebook.com/oauth/access_token', {
      client_id: this.clientId,
      client_secret: this.clientSecret,
      grant_type: 'client_credentials',
      scope: 'whatsapp_business_messaging'
    });
    
    const token = response.data.access_token;
    const expiresIn = response.data.expires_in;
    
    // Cache the token
    await this.tokenStorage.saveToken({
      token,
      expiresAt: Date.now() + (expiresIn * 1000)
    });
    
    return token;
  }
}
```

### 1.3 Webhook Configuration

- Create webhook endpoint in the application
- Implement verification challenge response
- Configure webhook subscriptions in Meta Dashboard
- Set up webhook signature verification

```typescript
// app/api/webhooks/whatsapp/route.ts
import { NextRequest, NextResponse } from "next/server";
import crypto from "crypto";

export async function GET(req: NextRequest) {
  // Handle the webhook verification challenge
  const { searchParams } = new URL(req.url);
  const mode = searchParams.get("hub.mode");
  const token = searchParams.get("hub.verify_token");
  const challenge = searchParams.get("hub.challenge");
  
  // Verify token should match what you set in the Meta Dashboard
  const VERIFY_TOKEN = process.env.WHATSAPP_WEBHOOK_VERIFY_TOKEN;
  
  if (mode === "subscribe" && token === VERIFY_TOKEN) {
    console.log("Webhook verified");
    return new NextResponse(challenge);
  } else {
    return NextResponse.json({ error: "Verification failed" }, { status: 403 });
  }
}

export async function POST(req: NextRequest) {
  // Get the request body
  const body = await req.json();
  
  // Verify signature
  const signature = req.headers.get("x-hub-signature-256") || "";
  const isValid = verifySignature(JSON.stringify(body), signature);
  
  if (!isValid) {
    return NextResponse.json({ error: "Invalid signature" }, { status: 403 });
  }
  
  // Process the webhook event
  if (body.object === "whatsapp_business_account") {
    // Handle different types of events
    if (body.entry && body.entry.length > 0) {
      for (const entry of body.entry) {
        for (const change of entry.changes) {
          if (change.field === "messages") {
            // Handle incoming messages
            await handleIncomingMessage(change.value);
          }
        }
      }
    }
    
    return NextResponse.json({ success: true });
  }
  
  return NextResponse.json({ error: "Unsupported webhook