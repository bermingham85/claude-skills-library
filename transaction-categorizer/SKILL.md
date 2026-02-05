---
name: transaction-categorizer
description: Categorizes financial transactions for Irish sole trader business. Applies vendor rules, Irish VAT rates, and routes to QuickBooks/Booke.AI ready format via n8n webhook.
---

# Transaction Categorizer Skill

Handles financial transaction categorization for Irish sole trader business (Airbnb hosting, subscriptions, business expenses).

## When to Activate

Activate when ANY of these occur:
- User shares bank statement (CSV, Excel)
- User mentions transactions, expenses, or receipts
- Keywords: QuickBooks, Booke.AI, categorize, reconcile, VAT
- User asks about business expenses or tax implications

## Irish VAT Rates

| Rate | Value | Apply To |
|------|-------|----------|
| Standard | 23% | Most services/goods, subscriptions |
| Reduced | 13.5% | Tourism, accommodation (Airbnb income) |
| Second Reduced | 9% | Utilities, newspapers |
| Exempt | N/A | Rent/lease |
| OOS | N/A | Transfers, cashback |

## Account Rules

- **AIB ending 1195**: Personal account - EXCLUDE from business
- **PayPal → Revolut Pro**: Internal transfer (non-taxable)
- **PayPal → AIB 1195**: Personal use of business funds
- **All Airbnb income**: VAT-inclusive at 13.5%
- **PayPal before June 2022**: Personal/pre-Airbnb

## Process

1. **Parse transactions** from provided data

2. **For each transaction**:
   - Match vendor against known rules
   - Apply category + VAT rate
   - Flag unknown vendors

3. **Call n8n webhook**:
   ```
   POST http://localhost:5678/webhook/transaction/categorize
   {
     "transactions": [
       {
         "date": "2026-02-05",
         "description": "PAYMENT - Google Ireland",
         "amount": -2.99,
         "account": "Revolut Pro"
       }
     ],
     "source": "revolut|paypal|aib"
   }
   ```

4. **Report results**: Categorized count, flagged items for review

## Known Vendors

| Pattern | Category | VAT |
|---------|----------|-----|
| Airbnb Payments Luxembourg | Airbnb Income | Reduced |
| Google Payment Ireland | Dues and subscriptions | Standard |
| Electric Ireland | Utilities | Second Reduced |
| Netflix, Spotify | Dues and subscriptions | Standard |
| ElevenLabs, OpenAI, Midjourney | Dues and subscriptions | Standard |
| Blink subscriptions | Internal Transfer | OOS |

## Category Mapping

| Expense Type | QuickBooks Category |
|-------------|---------------------|
| Subscriptions | Dues and subscriptions |
| Utilities | Utilities |
| Marketing/Ads | Advertising Expenses |
| Professional services | Legal and professional fees |
| Rent | Rent or lease payments |
| Equipment/Repairs | Repairs and Maintenance |

## Guidelines

- Auto-categorize recognized vendors immediately
- Flag unknown vendors for review (don't guess)
- Apply Irish VAT correctly
- Never categorize AIB 1195 as business
- Generate Booke.AI-compatible output format

## Example

```
User: "Here's my January Revolut statement" [CSV]

You:
1. Parse CSV
2. Match against known rules
3. POST to webhook
4. Say: "Categorized 45/50 transactions. 5 need review:
   - Unknown: STRIPE IRELAND (suggest: E-commerce expense @ 23%)
   - Unknown: ZOOM.US (suggest: Subscriptions @ 23%)"
```
