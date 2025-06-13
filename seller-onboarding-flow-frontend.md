# Seller Registration Guide

## Step 1: Sign In or Create an Account

- If you already have a buyer account on our platform, you can use it to register as a seller.  
- Otherwise, click **â€œCreate New Accountâ€** and enter:
  - **Full Name** (text field)
  - **Mobile Number** (number field)
  - **Email (Optional)** (email field)
  - **OTP for Verification** (otp input)
  - **Password** (password field)

---

## Step 2: Choose Business Type & Location

- Choose your **business type** and **location**:

**Business Types Supported:** (radio group with 4 options)
1. Individual Seller (No GST required for exempt products)  
2. Proprietorship  
3. Private Limited Company / LLP  
4. Others (NGOs, Co-ops, etc.)

**Location:** (radio button with India as the only option)
- Select **'India'**
- Enter your full legal name (text field)
- Click **â€œContinueâ€**

---

## Step 3: GST Details

Depending on your business type, youâ€™ll be prompted for:

- **GST Details** (Mandatory unless you're exempt):
  - **GSTIN** (auto-fills business name and address)

```js
// Function to fetch GSTIN details
async function fetchGSTINDetails(apiKey, gstin) {
  const url = `https://www.knowyourgst.com/developers/gstincall/?gstin=${gstin}`;
  try {
    const response = await fetch(url, {
      method: 'GET',
      headers: {
        'passthrough': apiKey
      }
    });

    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }

    const data = await response.json();
    console.log("GSTIN Details:", data);
    return data;
  } catch (error) {
    console.error("Error fetching GSTIN details:", error);
    return null;
  }
}

// Example usage:
const apiKey = "YOUR_API_KEY"; // Replace with your actual API key
const gstin = "06AADCH9716L1Z8"; // Replace with the GSTIN you want to look up
fetchGSTINDetails(apiKey, gstin);
```

**Example Response:**
```json
{
  "status_code": 1,
  "gstin": "06AADCH9716L1Z8",
  "legal-name": "HONASA CONSUMER LIMITED",
  "trade-name": "HONASA CONSUMER LIMITED",
  "pan": "AADCH9716L",
  "dealer-type": "Regular",
  "registration-date": "04/09/2018",
  "entity-type": "Public Limited Company",
  "business": "Office / Sale Office",
  "status": "Active",
  "adress": {
    "floor": "",
    "bno": "10th And 11th Floor, Capital Cyberscape",
    "bname": "Capital Cyberscape",
    "street": "Sector 59, Gurugram",
    "location": "Gurugram",
    "state": "Haryana",
    "lt": "28.4010000000001",
    "lg": "77.1033",
    "pincode": "122101",
    "city": "Gurgaon"
  }
}
```

- **Upload GST certificate** (file upload field)  
- **24-hour GST verification** before proceeding

> **Note:** If your products are GST-exempt (e.g., books, handmade goods), you can proceed without GST.

If you donâ€™t have a GST:
- Upload:
  - Company PAN (or Individual PAN)
  - File upload field

---

## Step 4: Customize Your Storefront

Customize your seller profile:
- **Store Name** (text field)
- **Store Description** (textarea)
- **Store Location** (text field)
- **Store Logo** (file upload)
- **Product Categories** (multi-select)
- **Brand Owner:** Yes / No

---

## Step 5: Pickup & Return Address

- **Shipping Type:**
  - **A)** You store, you ship (self or 3rd party courier)
  - **B)** You store, we ship (our courier)

- **Shipping Fee Preference:**
  - **A)** Provide free delivery (you pay shipping)
  - **B)** Add shipping charges for buyers

- **Default Pickup Address** (text field)  
- **Return Address** (text field with â€œsame as pickupâ€ checkbox)

---

## Step 6: Enter Bank Account Details

To receive payments:
- **Account Holderâ€™s Name** (text field)
- **Account Number** (text field)
- **IFSC Code** (text field)
- **(Optional)** Upload Cancelled Cheque or Bank Statement (file upload)
- **GST Rate** (dropdown field)

---

## Step 7: KYC Verification

Upload **any one** of the following with your photo (file upload):

- PAN Card  
- Aadhaar  
- Driving License  
- Voter ID

---

## Final Step: Confirm Tax & Legal Info

- Reconfirm **GST Details**
- Agree to **TCS Compliance Terms** (Tax Collected at Source)
- Accept **Terms of Service** for sellers