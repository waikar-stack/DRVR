# Fleet Ledger v2: data model and access

Version 1 runs on claude.ai for the family fleet and for testing. Version 2 is the standalone app: hosted on GitHub Pages, with logins and data on Firebase (Authentication and Firestore). Records inside a fleet keep v1's shape, so v1 data can be exported and imported.

## Who's who

| Person | Where their access is stored | Can see | Can change |
|---|---|---|---|
| Admin | `orgs/{org}/members/{uid}`, role `admin` | Every fleet in the organisation | Everything, including staff |
| Manager | role `manager` | Every fleet | All fleet data, owners and drivers, approvals |
| Clerk (data entry) | role `clerk` | Every fleet | Adds and edits payments, expenses, off-road days and mileage; cannot delete or approve |
| Owner | `orgs/{org}/fleets/{fleet}/access/{uid}`, role `owner` | Their own fleet only | Sends expenses for approval; withdraws them while pending |
| Owner's family | role `owner_viewer` | Their fleet only | Nothing |
| Driver | role `driver` with `driverId` | Their own statement and driver record | Nothing |

An organisation is a management company, or a family business running its own cars. Your family fleet becomes one organisation with one fleet: you are admin, and family members are managers or clerks.

## Structure

```
users/{uid}                              personal profile (only the person themselves)
invites/{code}                           invite codes (random, 20+ characters, expire within 15 days)
orgs/{orgId}                             name, createdBy
  members/{uid}                          staff: role admin | manager | clerk
  fleets/{fleetId}                       one per owner: name, summary figures
    access/{uid}                         uid, role owner | owner_viewer | driver, driverId
    settings/main                        rates, warning days, tracking start
    cars/{carId}
    drivers/{driverId}
    assignments/{id}                     car + driver + rate + rest day + start/end
    months/{YYYY-MM}/payments/{id}       M-Pesa receipt number as the ID (no double imports)
    months/{YYYY-MM}/expenses/{id}
    months/{YYYY-MM}/downtime/{id}
    months/{YYYY-MM}/odo/{id}
    months/{YYYY-MM}/activity/{id}       who did what (append only)
    months/{YYYY-MM}/visits/{uid_date}   who opened the app each day
    submissions/{id}                     owner expenses: pending → approved | rejected
    statements/{driverId}                each driver's balance and recent lines
```

## How people get in

Nobody can add themselves to anything. Every join goes through an invite code, redeemed in one batch that creates the access record and marks the code used, so each code works once.

1. **Founding:** a new user creates an organisation and their own admin record in one batch.
2. **Staff:** an admin makes a `manager` or `clerk` invite; the person signs up and redeems it.
3. **Owners:** a manager makes an `owner` (or `owner_viewer`) invite for one fleet and sends the link by WhatsApp or SMS.
4. **Drivers:** a manager makes a `driver` invite tied to one driver record; it can't link any other driver.

Managers can remove anyone's access at any time by deleting their access document.

## Owner expense approval

1. The owner creates `submissions/{id}` with `status: pending`. The rules check that the car is in their fleet and that the amount and date are valid.
2. A manager reviews it, corrects details if needed, and approves or declines. The rules allow only the decision fields to change, and only once.
3. On approval the app writes the expense to `months/{YYYY-MM}/expenses` with the ID `owner_{submissionId}`, so approving twice can't double-count.

## Driver statements

Drivers never read raw payments. Staff (later a Cloud Function) write one `statements/{driverId}` document per driver with the balance and recent transactions, and each driver reads only their own.

## Notes

- **Read costs:** each rule check that looks up a membership or access document counts as one extra read. At 10–50 cars this stays far inside the free tier.
- **Secrets:** the Firebase web config (apiKey, project ID) is safe in a public repo; the rules are what protect the data. Never commit service-account keys or Daraja credentials (see `.gitignore`).
- **Later, on the Blaze plan:** a Cloud Function for the Daraja C2B callback (plate as account number), and one that rebuilds driver statements and fleet summaries automatically.

## Records the app keeps for others

- **Fleet summary:** when a manager opens a fleet, the app saves this month's figures on the fleet document, which feeds the all-fleets overview.
- **Driver statements:** when staff open a fleet, the app refreshes each driver's `statements/{driverId}` document. A driver's page shows the latest one.
