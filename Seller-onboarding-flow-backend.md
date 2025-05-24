# Seller onboarding flow (backend)

## Table of Contents

1. [Overview & Base URL](#overview--base-url)
2. [Schemas (Zod)](#schemas-zod)
3. [Endpoints](#endpoints)

- 1. [Business Info](#1-business-info)
- 2. [GST Details](#2-gst-details)
- 3. [Storefront Setup](#3-storefront-setup)
- 4. [Shipping & Addresses](#4-shipping--addresses)
- 5. [Bank Details](#5-bank-details)
- 6. [KYC Verification](#6-kyc-verification)
- 7. [Legal Confirmation](#7-legal-confirmation)
- 8. [All-in-One Complete](#8-all-in-one-complete)
- 9. [Get Onboarding Progress](#9-get-onboarding-progress)
- 10. [GSTIN Lookup](#10-gstin-lookup)
- 11. [IFSC Code Lookup](#11-ifsc-code-lookup)

4. [Common Responses](#common-responses)
5. [Notes & Best Practices](#notes--best-practices)

---

## Overview & Base URL

- **Base URL**: `http://localhost:5000/api/v1/seller`
- **Content-Type**: `application/json`
- **Authentication**: All `/onboarding/*` and `/lookup/*` endpoints require a valid **HttpOnly** `accessToken` cookie.

---

## Schemas (Zod)

```ts
import * as z from "zod";

// Standard error response
export const errorSchema = z.object({
  error: z.enum([
    "Unauthorized",
    "BadRequest",
    "NotFound",
    "RateLimit",
    "ServerError",
  ]),
  message: z.string(),
});

// Step 1: Business Info
export const businessInfoSchema = z.object({
  businessType: z.enum([
    "individual",
    "proprietorship",
    "private_limited",
    "others",
  ]),
  country: z.literal("india"),
  legalName: z.string().min(3),
});

// Step 2: GST Details
export const gstSchema = z.object({
  gstin: z
    .string()
    .regex(
      /^[0-9]{2}[A-Z]{5}[0-9]{4}[A-Z][1-9A-Z]Z[0-9A-Z]$/,
      "Invalid GSTIN format"
    )
    .optional()
    .or(z.literal("")),
  withoutGst: z.boolean().optional(),
  exemptionReason: z.string().min(1, "Exemption reason is required").optional(),
  gstCertificate: z.string().optional(),
  panCard: z.string().optional(),
});

// Step 3: Storefront Setup
export const storefrontSchema = z.object({
  storeName: z.string().min(3, "Store name must be at least 3 characters"),
  storeDescription: z
    .string()
    .min(20, "Description must be at least 20 characters")
    .max(500, "Description cannot exceed 500 characters"),
  storeLocation: z.string().min(3, "Location must be at least 3 characters"),
  storeLogo: z.string().optional(),
  productCategories: z.array(z.string()).min(1, "Select at least one category"),
  isBrandOwner: z.boolean(),
});

// Step 4: Shipping & Addresses
export const shippingSchema = z.object({
  shippingType: z.enum(["self", "shiprocket"]),
  shippingFee: z.enum(["free", "paid"]),
  pickupAddress: z.object({
    addressLine1: z.string().min(3, "Address must be at least 3 characters"),
    addressLine2: z.string().optional(),
    city: z.string().min(2, "City is required"),
    state: z.string().min(2, "State is required"),
    pincode: z.string().regex(/^[1-9][0-9]{5}$/, "Invalid pincode"),
  }),
  returnAddressSameAsPickup: z.boolean(),
  returnAddress: z
    .object({
      addressLine1: z.string().min(3, "Address must be at least 3 characters"),
      addressLine2: z.string().optional(),
      city: z.string().min(2, "City is required"),
      state: z.string().min(2, "State is required"),
      pincode: z.string().regex(/^[1-9][0-9]{5}$/, "Invalid pincode"),
    })
    .optional(),
  packageDetails: z
    .object({
      weight: z.string().min(1, "Weight is required"),
      length: z.string().min(1, "Length is required"),
      width: z.string().min(1, "Width is required"),
      height: z.string().min(1, "Height is required"),
    })
    .optional(),
});

// Step 5: Bank Details
export const bankSchema = z.object({
  accountName: z.string().min(3, "Account name must be at least 3 characters"),
  accountNumber: z
    .string()
    .regex(/^\d{9,18}$/, "Account number must be between 9 and 18 digits"),
  ifscCode: z
    .string()
    .regex(/^[A-Za-z]{4}0[A-Z0-9]{6}$/, "Invalid IFSC code format"),
  bankDocument: z.string().optional(),
});

// Step 6: KYC Verification
export const kycSchema = z.object({
  documentType: z.enum(["pan", "aadhar", "driving_license", "voter_id"]),
  document: z.string().min(1, "Document upload is required"),
  selfie: z.string().min(1, "Selfie upload is required"),
});

// Step 7: Legal Confirmation
export const legalSchema = z.object({
  tcsCompliance: z.boolean().refine((v) => v === true, {
    message: "You must agree to TCS compliance terms",
  }),
  termsOfService: z.boolean().refine((v) => v === true, {
    message: "You must agree to Terms of Service",
  }),
});

// Combined for All-in-One
export const sellerOnboardingSchema = z.object({
  businessInfo: businessInfoSchema,
  gst: gstSchema,
  storefront: storefrontSchema,
  shipping: shippingSchema,
  bank: bankSchema,
  kyc: kycSchema,
  legal: legalSchema,
});

// Progress Tracker
export const progressSchema = z.object({
  completed: z.array(
    z.enum([
      "businessInfo",
      "gst",
      "storefront",
      "shipping",
      "bank",
      "kyc",
      "legal",
    ])
  ),
  left: z.array(
    z.enum([
      "businessInfo",
      "gst",
      "storefront",
      "shipping",
      "bank",
      "kyc",
      "legal",
    ])
  ),
  current: z.enum([
    "businessInfo",
    "gst",
    "storefront",
    "shipping",
    "bank",
    "kyc",
    "legal",
    "done",
  ]),
});

// GST Lookup Result
export const gstLookupSchema = z.object({
  gstin: z.string(),
  "legal-name": z.string(),
  "trade-name": z.string().optional(),
  pan: z.string(),
  "dealer-type": z.string(),
  "registration-date": z.string(),
  "entity-type": z.string(),
  business: z.string(),
  status: z.string(),
  address: z.object({
    floor: z.string().optional(),
    bno: z.string(),
    bname: z.string().optional(),
    street: z.string(),
    location: z.string(),
    city: z.string(),
    state: z.string(),
    pincode: z.string(),
    lt: z.string().optional(),
    lg: z.string().optional(),
  }),
});

// IFSC Lookup Result
export const ifscLookupSchema = z.object({
  ifsc: z.string(),
  bankName: z.string(),
  branch: z.string(),
  address: z.string(),
  micr: z.string(),
  contact: z.string().optional(),
});

export type SellerOnboardingData = z.infer<typeof sellerOnboardingSchema>;
export type OnboardingProgress = z.infer<typeof progressSchema>;
export type GstLookupResult = z.infer<typeof gstLookupSchema>;
export type IfscLookupResult = z.infer<typeof ifscLookupSchema>;
export type ErrorResponse = z.infer<typeof errorSchema>;
```

---

## Endpoints

### 1. Business Info

**PUT** `/onboarding/business-info`

```http
Cookie: accessToken=<jwt>
Content-Type: application/json
```

**Request Body** (matches `businessInfoSchema`)

```json
{
  "businessType": "private_limited",
  "country": "india",
  "legalName": "Alice Retail Pvt Ltd"
}
```

**Responses**

- `200 OK`

  ```json
  { "message": "Business info saved." }
  ```

- _See [Common Responses](#common-responses)_

---

### 2. GST Details

**PUT** `/onboarding/gst`

```http
Cookie: accessToken=<jwt>
Content-Type: application/json
```

**Request Body** (matches `gstSchema`)

```json
{
  "gstin": "27ABCDE1234F2Z5",
  "withoutGst": false,
  "exemptionReason": "",
  "gstCertificate": "<file-id>",
  "panCard": "<file-id>"
}
```

> If `withoutGst=true`, `exemptionReason` and `panCard` are required.

**Responses**

- `200 OK`

  ```json
  { "message": "GST details saved." }
  ```

- _See [Common Responses](#common-responses)_

---

### 3. Storefront Setup

**PUT** `/onboarding/storefront`

```http
Cookie: accessToken=<jwt>
Content-Type: application/json
```

**Request Body** (matches `storefrontSchema`)

```json
{
  "storeName": "Alice’s Boutique",
  "storeDescription": "Handcrafted bags & wallets made in India.",
  "storeLocation": "Mumbai, MH",
  "storeLogo": "<file-id>",
  "productCategories": ["bags", "wallets"],
  "isBrandOwner": true
}
```

**Response**

- `200 OK`

  ```json
  { "message": "Storefront configured." }
  ```

- _See [Common Responses](#common-responses)_

---

### 4. Shipping & Addresses

**PUT** `/onboarding/shipping`

```http
Cookie: accessToken=<jwt>
Content-Type: application/json
```

**Request Body** (matches `shippingSchema`)

```json
{
  "shippingType": "self",
  "shippingFee": "paid",
  "pickupAddress": {
    "addressLine1": "123 Market St",
    "city": "Mumbai",
    "state": "Maharashtra",
    "pincode": "400001"
  },
  "returnAddressSameAsPickup": false,
  "returnAddress": {
    "addressLine1": "Warehouse 5",
    "city": "Mumbai",
    "state": "Maharashtra",
    "pincode": "400002"
  },
  "packageDetails": {
    "weight": "1kg",
    "length": "10cm",
    "width": "5cm",
    "height": "3cm"
  }
}
```

- Omit `returnAddress` if `returnAddressSameAsPickup=true`.
- Omit `packageDetails` if unused.

**Responses**

- `200 OK`

  ```json
  { "message": "Shipping info saved." }
  ```

- _See [Common Responses](#common-responses)_

---

### 5. Bank Details

**PUT** `/onboarding/bank`

```http
Cookie: accessToken=<jwt>
Content-Type: application/json
```

**Request Body** (matches `bankSchema`)

```json
{
  "accountName": "Alice Patel",
  "accountNumber": "123456789012",
  "ifscCode": "HDFC0001234",
  "bankDocument": "<file-id>"
}
```

**Responses**

- `200 OK`

  ```json
  { "message": "Bank details saved." }
  ```

- _See [Common Responses](#common-responses)_

---

### 6. KYC Verification

**PUT** `/onboarding/kyc`

```http
Cookie: accessToken=<jwt>
Content-Type: application/json
```

**Request Body** (matches `kycSchema`)

```json
{
  "documentType": "pan",
  "document": "<file-id>",
  "selfie": "<file-id>"
}
```

**Responses**

- `200 OK`

  ```json
  { "message": "KYC submitted." }
  ```

- _See [Common Responses](#common-responses)_

---

### 7. Legal Confirmation

**PUT** `/onboarding/legal`

```http
Cookie: accessToken=<jwt>
Content-Type: application/json
```

**Request Body** (matches `legalSchema`)

```json
{
  "tcsCompliance": true,
  "termsOfService": true
}
```

**Responses**

- `200 OK`

  ```json
  { "message": "Legal terms accepted." }
  ```

- _See [Common Responses](#common-responses)_

---

### 8. All-in-One Complete

**POST** `/onboarding/complete`

```http
Cookie: accessToken=<jwt>
Content-Type: application/json
```

**Request Body** (matches `sellerOnboardingSchema`)

```json
{
  "businessInfo":  { … },
  "gst":           { … },
  "storefront":    { … },
  "shipping":      { … },
  "bank":          { … },
  "kyc":           { … },
  "legal":         { … }
}
```

**Responses**

- `200 OK`

  ```json
  { "message": "Onboarding complete." }
  ```

- `400 Bad Request` →

  ```json
  {
    "error": "BadRequest",
    "message": "Validation failed",
    "details": [
      /* per-step Zod errors */
    ]
  }
  ```

- `401 Unauthorized` → `ErrorResponse`

---

### 9. Get Onboarding Progress

**GET** `/onboarding/progress`

```http
Cookie: accessToken=<jwt>
```

**Response** (matches `progressSchema`)

```json
{
  "completed": ["businessInfo", "gst"],
  "left": ["storefront", "shipping", "bank", "kyc", "legal"],
  "current": "storefront"
}
```

- `200 OK`
- `401 Unauthorized` → `ErrorResponse`

---

### 10. GSTIN Lookup

**GET** `/lookup/gst/:gstin`

```http
Cookie: accessToken=<jwt>
```

**Path Parameter**

- `:gstin` — 15-character GSTIN

**Response** (matches `gstLookupSchema`)

```json
{
  "gstin": "06AADCH9716L1Z8",
  "legal-name": "HONASA CONSUMER LIMITED",
  "trade-name": "HONASA CONSUMER LIMITED",
  "pan": "AADCH9716L",
  "dealer-type": "Regular",
  "registration-date": "04/09/2018",
  "entity-type": "Public Limited Company",
  "business": "Office / Sale Office",
  "status": "Active",
  "address": {
    "floor": "",
    "bno": "10th And 11th Floor, Capital Cyberscape",
    "bname": "Capital Cyberscape",
    "street": "Sector 59, Gurugram",
    "location": "Gurugram",
    "city": "Gurgaon",
    "state": "Haryana",
    "pincode": "122101",
    "lt": "28.4010000000001",
    "lg": "77.1033"
  }
}
```

- `200 OK`
- _See [Common Responses](#common-responses)_
- `404 Not Found` → `ErrorResponse`

---

### 11. IFSC Code Lookup

**GET** `/lookup/ifsc/:ifscCode`

```http
Cookie: accessToken=<jwt>
```

**Path Parameter**

- `:ifscCode` — 11-character IFSC

**Response** (matches `ifscLookupSchema`)

```json
{
  "ifsc": "HDFC0001234",
  "bankName": "HDFC Bank Ltd",
  "branch": "Connaught Place",
  "address": "Concourse Level, CP Tower, Connaught Place, New Delhi, 110001",
  "micr": "110240002",
  "contact": "+91-11-23456789"
}
```

- `200 OK`
- _See [Common Responses](#common-responses)_
- `404 Not Found` → `ErrorResponse`

---

## Common Responses

| HTTP Code | Schema        | Description                   |
| --------- | ------------- | ----------------------------- |
| 400       | ErrorResponse | Validation or bad input       |
| 401       | ErrorResponse | Missing/expired `accessToken` |
| 404       | ErrorResponse | Resource not found            |
| 429       | ErrorResponse | Rate limit exceeded           |
| 500       | ErrorResponse | Server-side error             |

---

## Notes & Best Practices

- **File uploads** (logos, docs): POST to file-service, then use returned file IDs.
- **CSRF**: Use `SameSite=Strict`; if relaxed, implement double-submit tokens.
- **Session expiry**: Blacklist refresh tokens server-side for inactivity.
- **Rate-limiting**: Protect all endpoints (e.g. 100 req/hr per IP).
- **Caching**: Cache GST/IFSC lookup results (TTL \~24 hrs).
- **Monitoring**: Alert on >5% upstream errors over 5 min or latency spikes.
- **Versioning**: All v1 endpoints live under `/v1/...`. Breaking changes will go into a v2, leaving v1 supported until at least 2026-01-01.
