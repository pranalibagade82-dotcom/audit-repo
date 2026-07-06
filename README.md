[ Start Script ]
              │
              ▼
    Prompt for GitHub Token
              │
              ▼
   Fetch All Repositories 
     from Organization
              │
              ▼
    Is API Response 200 OK? ──(No)──► [ Print Error & Exit ]
              │
            (Yes)
              │
              ▼
┌───────────────────────────┐
│ Loop Through Each Repo    │◄───────────────────────────────────┐
└─────────────┬─────────────┘                                    │
              │                                                  │
              ▼                                                  │
     Is Repo Excluded? ──────────(Yes)──► [ Mark SKIPPED ] ──────┤
              │                                                  │
             (No)                                                │
              │                                                  │
              ▼                                                  │
    Fetch Branch Protection                                      │
           Settings                                              │
              │                                                  │
              ▼                                                  │
   Does Protection Exist? ───────(No)──► [ Mark FAIL ] ──────────┤
              │                          (Add to Queue)          │
            (Yes)                                                │
              │                                                  │
              ▼                                                  │
 Is PR Review Rule Enforced?                                     │
        │             │                                          │
      (Yes)          (No)                                        │
        │             │                                          │
        ▼             ▼                                          │
  [ Mark PASS ]  [ Mark FAIL ] ──────────────────────────────────┘
                 (Add to Queue)
                      │
                      ▼
             (After Last Repo)
                      │
                      ▼
           [ Print Summary Table ]
                      │
                      ▼
           [ Return Failed Queue ]
