# _github-pages-challenge-charbash07-png
Arbash referral demo.
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Arbash Referral Demo</title>
  <style>
    body{font-family:Inter,system-ui,Segoe UI,Roboto,Arial;margin:0;background:#f7f7fb;color:#111}
    .wrap{max-width:900px;margin:28px auto;padding:20px}
    header{display:flex;align-items:center;gap:14px}
    header h1{margin:0;font-size:20px}
    .card{background:white;border-radius:12px;padding:18px;margin-top:14px;box-shadow:0 6px 20px rgba(10,12,20,0.06)}
    button{cursor:pointer;border:0;padding:10px 14px;border-radius:8px;background:#111;color:white}
    input[type=text]{padding:10px;border-radius:8px;border:1px solid #ddd;width:100%}
    .small{font-size:13px;color:#666;margin-top:6px}
    .owner-badge{display:inline-block;background:#ffd33d;color:#111;padding:6px 10px;border-radius:999px;font-weight:600}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div style="width:56px;height:56px;border-radius:12px;background:#111;color:#fff;display:flex;align-items:center;justify-content:center;font-weight:700">ADZ</div>
      <div>
        <h1>Arbash Referral Demo <span style="font-size:12px;color:#777">(Free GitHub Pages)</span></h1>
        <div class="small">Owner auto-recognition via Gmail: <b>charbash07@gmail.com</b></div>
      </div>
    </header>

    <div id="authCard" class="card">
      <div id="signedOut">
        <p><b>Sign in</b> (Google)</p>
        <button id="googleSign">Sign in with Google</button>
        <div class="small">Use your Google account. If you are the owner (charbash07@gmail.com) you will see the Owner Panel.</div>
      </div>

      <div id="signedIn" style="display:none">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <div>
            <div id="userName" style="font-weight:700"></div>
            <div id="userEmail" class="small"></div>
          </div>
          <div>
            <span id="ownerBadge" class="owner-badge" style="display:none">OWNER</span>
            <button id="signOut" style="margin-left:10px">Sign out</button>
          </div>
        </div>
      </div>
    </div>

    <div id="ownerPanel" class="card" style="display:none">
      <h3>Owner Dashboard</h3>
      <div class="small">Welcome, Owner. From here you will later review withdrawals, top referrers and campaign controls.</div>
      <div style="margin-top:12px">
        <button id="simulateReport">Simulate Report (demo)</button>
      </div>
      <pre id="reportArea" style="margin-top:12px;background:#f5f6fb;padding:12px;border-radius:8px;display:none"></pre>
    </div>

    <div id="userPanel" class="card" style="display:none">
      <h3>Your Referral Panel</h3>
      <div>
        <label>Referral link (share this):</label>
        <div style="display:flex;gap:8px;margin-top:8px">
          <input id="refLink" type="text" readonly />
          <button id="copyBtn">Copy</button>
        </div>
        <div class="small" style="margin-top:8px">Share on WhatsApp, Telegram, or social. When friend signs up using this link, you'll earn (demo).</div>
      </div>

      <div style="margin-top:14px">
        <h4>Wallet (demo)</h4>
        <div style="font-size:20px;font-weight:700">₹<span id="walletAmt">0.00</span></div>
        <div class="small" style="margin-top:6px">Withdrawals and real payouts require backend & KYC — this is a demo frontend.</div>
      </div>
    </div>

    <div class="card" style="margin-top:16px">
      <b>How to finish setup (Firebase Google Login)</b>
      <ol class="small">
        <li>Create a Firebase project at <b>console.firebase.google.com</b>, add a Web app and copy the config into the file where indicated.</li>
        <li>In Firebase → Authentication → Sign-in method → Enable Google.</li>
        <li>Add your GitHub Pages domain (e.g. <code>youruser.github.io</code>) under Authorized domains in Firebase Auth.</li>
        <li>Commit the file and open your GitHub Pages URL. Click Sign in → choose account.</li>
      </ol>
    </div>
  </div>

  <!-- Firebase + app logic -->
  <script type="module">
    // ---------- CONFIG: paste your Firebase web config here ----------
    // Replace the values below with your Firebase project's config object
    const firebaseConfig = {
      apiKey: "PASTE_API_KEY",
      authDomain: "PASTE_PROJECT.firebaseapp.com",
      projectId: "PASTE_PROJECT",
      storageBucket: "PASTE_PROJECT.appspot.com",
      messagingSenderId: "PASTE_ID",
      appId: "PASTE_APP_ID"
    };
    // -----------------------------------------------------------------

    // Owner email (auto-recognize)
    const OWNER_EMAIL = "charbash07@gmail.com";

    // minimal firebase setup using modular SDK
    import { initializeApp } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-app.js";
    import {
      getAuth,
      GoogleAuthProvider,
      signInWithPopup,
      signOut,
      onAuthStateChanged
    } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-auth.js";

    const app = initializeApp(firebaseConfig);
    const auth = getAuth(app);
    const provider = new GoogleAuthProvider();

    // UI refs
    const googleSign = document.getElementById('googleSign');
    const signOutBtn = document.getElementById('signOut');
    const signedOut = document.getElementById('signedOut');
    const signedIn = document.getElementById('signedIn');
    const userNameEl = document.getElementById('userName');
    const userEmailEl = document.getElementById('userEmail');
    const ownerBadge = document.getElementById('ownerBadge');
    const ownerPanel = document.getElementById('ownerPanel');
    const userPanel = document.getElementById('userPanel');
    const refLinkEl = document.getElementById('refLink');
    const copyBtn = document.getElementById('copyBtn');
    const walletAmt = document.getElementById('walletAmt');
    const reportArea = document.getElementById('reportArea');
    const simulateReport = document.getElementById('simulateReport');

    googleSign.addEventListener('click', () => {
      signInWithPopup(auth, provider).catch(e => alert('Login error: ' + e.message));
    });

    signOutBtn.addEventListener('click', () => signOut(auth).catch(e => alert(e.message)));

    onAuthStateChanged(auth, user => {
      if (user) {
        signedOut.style.display = 'none';
        signedIn.style.display = 'block';
        userNameEl.textContent = user.displayName || 'User';
        userEmailEl.textContent = user.email || '';
        // owner check
        if (user.email && user.email.toLowerCase() === OWNER_EMAIL.toLowerCase()) {
          ownerBadge.style.display = 'inline-block';
          ownerPanel.style.display = 'block';
          userPanel.style.display = 'none';
        } else {
          ownerBadge.style.display = 'none';
          ownerPanel.style.display = 'none';
          userPanel.style.display = 'block';
          // generate a simple referral link (demo uses user.uid)
          const refId = user.uid || encodeURIComponent(user.email || 'guest');
          const link = `${location.origin}${location.pathname}?ref=${encodeURIComponent(refId)}`;
          refLinkEl.value = link;
          // wallet demo: store balance in localStorage per user.email
          const key = 'wallet_' + (user.email || refId);
          const current = Number(localStorage.getItem(key) || 0);
          walletAmt.textContent = (current/100).toFixed(2);
        }
      } else {
        signedOut.style.display = 'block';
        signedIn.style.display = 'none';
        ownerPanel.style.display = 'none';
        userPanel.style.display = 'none';
      }
    });

    copyBtn.addEventListener('click', () => {
      navigator.clipboard.writeText(refLinkEl.value).then(()=> alert('Copied referral link'));
    });

    // simulate owner report (demo)
    simulateReport.addEventListener('click', () => {
      reportArea.style.display = 'block';
      reportArea.textContent = 'Top referrers:\\n1) userA - 120\\n2) userB - 88\\n3) userC - 45';
    });

    // When a visitor opens with ?ref=ID we can show a tiny hint (demo)
    const urlRef = new URL(location.href).searchParams.get('ref');
    if (urlRef) {
      // for demo we only show console; in production you would record this to Firestore/DB
      console.log('Visitor has ref param:', urlRef);
    }
  </script>
</body>
</html>
