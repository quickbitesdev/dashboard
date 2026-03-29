# Wise API Guide

## Credentials
- API Token: environment variable `WISE_API_TOKEN`
- Profile ID: environment variable `WISE_PROFILE_ID`
- Base URL: `https://api.wise.com`

## Working Endpoints

### Check Balance
```bash
curl -H "Authorization: Bearer $WISE_API_TOKEN" \
  "https://api.wise.com/v4/profiles/$WISE_PROFILE_ID/balances?types=STANDARD"
```

### List Recipients
```bash
curl -H "Authorization: Bearer $WISE_API_TOKEN" \
  "https://api.wise.com/v1/accounts?profile=$WISE_PROFILE_ID"
```

### Create Quote (for transfers)
```bash
curl -X POST -H "Authorization: Bearer $WISE_API_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.wise.com/v3/profiles/$WISE_PROFILE_ID/quotes" \
  -d '{"sourceCurrency": "EUR", "targetCurrency": "EUR", "sourceAmount": 5.0}'
```

## Limitations
- Card API (`/v3/spend/`) returns 403 — token doesn't have card management permissions
- Virtual card details must be accessed via Wise web UI or app
- Card number: check Discord chat history or Wise app

## Virtual Card
- Card exists, created via web UI
- Use for online purchases (Mullvad VPN etc)
- Cannot programmatically get card number via API
