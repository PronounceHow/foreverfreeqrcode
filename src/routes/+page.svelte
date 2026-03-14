<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import QRCodeStyling from 'qr-code-styling';

  // --- Types ---
  type TabId = 'website' | 'text' | 'wifi' | 'vcard' | 'appstore' | 'email' | 'phone' | 'sms';
  type DotType = 'square' | 'dots' | 'rounded' | 'classy' | 'classy-rounded' | 'extra-rounded';
  type CornerType = 'square' | 'extra-rounded' | 'dot';

  // --- State ---
  let activeTab: TabId = 'website';
  let qrInstance: QRCodeStyling | null = null;
  let qrContainer: HTMLDivElement;
  let logoFile: File | null = null;
  let logoDataUrl: string = '';
  let dotColor = '#000000';
  let bgColor = '#ffffff';
  let dotType: DotType = 'square';
  let cornerType: CornerType = 'square';
  let qrSize = 300;

  // Tab data
  let websiteUrl = 'https://';
  let plainText = '';
  let wifiSsid = '';
  let wifiPassword = '';
  let wifiEncryption = 'WPA';
  let wifiHidden = false;
  let vcardFirstName = '';
  let vcardLastName = '';
  let vcardPhone = '';
  let vcardEmail = '';
  let vcardOrg = '';
  let vcardUrl = '';
  let vcardAddress = '';
  let appIos = '';
  let appAndroid = '';
  let emailTo = '';
  let emailSubject = '';
  let emailBody = '';
  let phoneNumber = '';
  let smsNumber = '';
  let smsMessage = '';

  const tabs: { id: TabId; label: string; icon: string }[] = [
    { id: 'website', label: 'Website', icon: '🌐' },
    { id: 'vcard', label: 'Business Card', icon: '👤' },
    { id: 'wifi', label: 'WiFi', icon: '📶' },
    { id: 'appstore', label: 'App Store', icon: '📱' },
    { id: 'text', label: 'Text', icon: '📝' },
    { id: 'sms', label: 'SMS', icon: '💬' },
    { id: 'email', label: 'Email', icon: '✉️' },
    { id: 'phone', label: 'Phone', icon: '📞' },
  ];

  function buildQRData(): string {
    switch (activeTab) {
      case 'website':
        return websiteUrl || 'https://example.com';
      case 'text':
        return plainText || 'Hello World';
      case 'wifi':
        return `WIFI:T:${wifiEncryption};S:${wifiSsid};P:${wifiPassword};H:${wifiHidden ? 'true' : 'false'};;`;
      case 'vcard':
        return [
          'BEGIN:VCARD',
          'VERSION:3.0',
          `N:${vcardLastName};${vcardFirstName}`,
          `FN:${vcardFirstName} ${vcardLastName}`,
          vcardOrg ? `ORG:${vcardOrg}` : '',
          vcardPhone ? `TEL:${vcardPhone}` : '',
          vcardEmail ? `EMAIL:${vcardEmail}` : '',
          vcardUrl ? `URL:${vcardUrl}` : '',
          vcardAddress ? `ADR:;;${vcardAddress}` : '',
          'END:VCARD',
        ].filter(Boolean).join('\n');
      case 'appstore':
        if (appIos && appAndroid) return appIos; // default to iOS; user can pick
        return appIos || appAndroid || 'https://apps.apple.com';
      case 'email':
        const params = new URLSearchParams();
        if (emailSubject) params.set('subject', emailSubject);
        if (emailBody) params.set('body', emailBody);
        const qs = params.toString();
        return `mailto:${emailTo}${qs ? '?' + qs : ''}`;
      case 'phone':
        return `tel:${phoneNumber}`;
      case 'sms':
        return `sms:${smsNumber}${smsMessage ? '?body=' + encodeURIComponent(smsMessage) : ''}`;
      default:
        return '';
    }
  }

  function getQROptions() {
    return {
      width: qrSize,
      height: qrSize,
      data: buildQRData(),
      image: logoDataUrl || undefined,
      dotsOptions: { color: dotColor, type: dotType },
      backgroundOptions: { color: bgColor },
      cornersSquareOptions: { type: cornerType },
      imageOptions: { crossOrigin: 'anonymous', margin: 6, imageSize: 0.3 },
      qrOptions: { errorCorrectionLevel: logoDataUrl ? 'H' : 'M' as any },
    };
  }

  function renderQR() {
    if (!qrContainer) return;
    qrContainer.innerHTML = '';
    qrInstance = new QRCodeStyling(getQROptions());
    qrInstance.append(qrContainer);
  }

  // Explicitly list every reactive var so Svelte re-runs when any changes
  $: activeTab, dotColor, bgColor, dotType, cornerType, qrSize, logoDataUrl,
     websiteUrl, plainText, wifiSsid, wifiPassword, wifiEncryption, wifiHidden,
     vcardFirstName, vcardLastName, vcardPhone, vcardEmail, vcardOrg, vcardUrl, vcardAddress,
     appIos, appAndroid, emailTo, emailSubject, emailBody, phoneNumber, smsNumber, smsMessage,
     renderQR();

  function handleLogoUpload(e: Event) {
    const input = e.target as HTMLInputElement;
    const file = input.files?.[0];
    if (!file) return;
    logoFile = file;
    const reader = new FileReader();
    reader.onload = (ev) => {
      logoDataUrl = ev.target?.result as string;
    };
    reader.readAsDataURL(file);
  }

  function removeLogo() {
    logoFile = null;
    logoDataUrl = '';
  }

  async function downloadPNG() {
    if (!qrInstance) return;
    await qrInstance.download({ name: 'qrcode', extension: 'png' });
  }

  async function downloadSVG() {
    if (!qrInstance) return;
    await qrInstance.download({ name: 'qrcode', extension: 'svg' });
  }

  onMount(() => {
    renderQR();
  });
</script>

<!-- Header -->
<header>
  <div class="header-inner">
    <div class="logo">
      <img src="/favicon.svg" alt="ForeverFreeQRCode logo" class="logo-icon" />
      <div>
        <span class="logo-title">ForeverFree<span class="accent">QRCode</span></span>
        <span class="logo-sub">.org</span>
      </div>
    </div>
    <p class="tagline">Static QR codes that never expire — free forever, no login, no tricks.</p>
  </div>
</header>

<!-- Main -->
<main>
  <div class="app-grid">

    <!-- LEFT: Form -->
    <section class="form-panel">

      <!-- Tabs -->
      <div class="tabs" role="tablist">
        {#each tabs as tab}
          <button
            class="tab-btn"
            class:active={activeTab === tab.id}
            on:click={() => activeTab = tab.id}
            role="tab"
            aria-selected={activeTab === tab.id}
          >
            <span class="tab-icon">{tab.icon}</span>
            <span class="tab-label">{tab.label}</span>
          </button>
        {/each}
      </div>

      <!-- Tab content -->
      <div class="tab-content">

        {#if activeTab === 'website'}
          <label class="field">
            <span>URL</span>
            <input type="url" bind:value={websiteUrl} placeholder="https://yourwebsite.com" />
          </label>

        {:else if activeTab === 'text'}
          <label class="field">
            <span>Text
              <span class="char-count" class:warn={plainText.length > 300} class:error={plainText.length > 500}>
                {plainText.length} / 500
              </span>
            </span>
            <textarea
              bind:value={plainText}
              placeholder="Enter any text..."
              rows="4"
              maxlength="500"
            ></textarea>
          </label>
          {#if plainText.length > 300}
            <p class="hint warn-hint">
              {#if plainText.length > 500}
                ⛔ Text too long — some scanners will fail. Keep it under 500 characters.
              {:else}
                ⚠️ Long text makes the QR code dense. Under 300 characters scans most reliably.
              {/if}
            </p>
          {/if}

        {:else if activeTab === 'wifi'}
          <label class="field">
            <span>Network Name (SSID)</span>
            <input type="text" bind:value={wifiSsid} placeholder="MyWiFiNetwork" />
          </label>
          <label class="field">
            <span>Password</span>
            <input type="text" bind:value={wifiPassword} placeholder="password123" />
          </label>
          <label class="field">
            <span>Encryption</span>
            <select bind:value={wifiEncryption}>
              <option value="WPA">WPA/WPA2</option>
              <option value="WEP">WEP</option>
              <option value="nopass">None (open)</option>
            </select>
          </label>
          <label class="field checkbox">
            <input type="checkbox" bind:checked={wifiHidden} />
            <span>Hidden network</span>
          </label>

        {:else if activeTab === 'vcard'}
          <div class="field-row">
            <label class="field">
              <span>First Name</span>
              <input type="text" bind:value={vcardFirstName} placeholder="Jane" />
            </label>
            <label class="field">
              <span>Last Name</span>
              <input type="text" bind:value={vcardLastName} placeholder="Doe" />
            </label>
          </div>
          <label class="field">
            <span>Organization</span>
            <input type="text" bind:value={vcardOrg} placeholder="Acme Corp" />
          </label>
          <label class="field">
            <span>Phone</span>
            <input type="tel" bind:value={vcardPhone} placeholder="+1 555 000 0000" />
          </label>
          <label class="field">
            <span>Email</span>
            <input type="email" bind:value={vcardEmail} placeholder="jane@example.com" />
          </label>
          <label class="field">
            <span>Website</span>
            <input type="url" bind:value={vcardUrl} placeholder="https://janedoe.com" />
          </label>
          <label class="field">
            <span>Address</span>
            <input type="text" bind:value={vcardAddress} placeholder="123 Main St, City, Country" />
          </label>

        {:else if activeTab === 'appstore'}
          <label class="field">
            <span>🍎 iOS App Store URL</span>
            <input type="url" bind:value={appIos} placeholder="https://apps.apple.com/app/..." />
          </label>
          <label class="field">
            <span>🤖 Google Play URL</span>
            <input type="url" bind:value={appAndroid} placeholder="https://play.google.com/store/apps/..." />
          </label>
          {#if appIos && appAndroid}
            <p class="hint">Both entered — QR will link to the iOS App Store. For Android, remove the iOS URL.</p>
          {/if}

        {:else if activeTab === 'email'}
          <label class="field">
            <span>To (email address)</span>
            <input type="email" bind:value={emailTo} placeholder="someone@example.com" />
          </label>
          <label class="field">
            <span>Subject (optional)</span>
            <input type="text" bind:value={emailSubject} placeholder="Hello!" />
          </label>
          <label class="field">
            <span>Body (optional)
              <span class="char-count" class:warn={emailBody.length > 200} class:error={emailBody.length > 300}>
                {emailBody.length} / 300
              </span>
            </span>
            <textarea bind:value={emailBody} placeholder="Pre-filled message..." rows="3" maxlength="300"></textarea>
          </label>
          {#if emailBody.length > 200}
            <p class="hint warn-hint">
              {#if emailBody.length > 300}
                ⛔ Body too long — keep under 300 characters for reliable scanning.
              {:else}
                ⚠️ Long body makes the QR code dense. Under 200 characters is ideal.
              {/if}
            </p>
          {/if}

        {:else if activeTab === 'phone'}
          <label class="field">
            <span>Phone Number</span>
            <input type="tel" bind:value={phoneNumber} placeholder="+1 555 000 0000" />
          </label>

        {:else if activeTab === 'sms'}
          <label class="field">
            <span>Phone Number</span>
            <input type="tel" bind:value={smsNumber} placeholder="+1 555 000 0000" />
          </label>
          <label class="field">
            <span>Message (optional)
              <span class="char-count" class:warn={smsMessage.length > 120} class:error={smsMessage.length > 160}>
                {smsMessage.length} / 160
              </span>
            </span>
            <textarea bind:value={smsMessage} placeholder="Pre-filled SMS..." rows="3" maxlength="160"></textarea>
          </label>
          {#if smsMessage.length > 120}
            <p class="hint warn-hint">
              {#if smsMessage.length > 160}
                ⛔ Message too long — keep under 160 characters for reliable scanning.
              {:else}
                ⚠️ Standard SMS is 160 characters. Getting close to the limit.
              {/if}
            </p>
          {/if}
        {/if}

      </div>

      <!-- Customization -->
      <div class="customization">
        <h3>Customize</h3>

        <div class="custom-grid">
          <label class="field">
            <span>Dot color</span>
            <div class="color-row">
              <input type="color" bind:value={dotColor} />
              <input type="text" bind:value={dotColor} maxlength="7" class="color-hex" />
            </div>
          </label>

          <label class="field">
            <span>Background</span>
            <div class="color-row">
              <input type="color" bind:value={bgColor} />
              <input type="text" bind:value={bgColor} maxlength="7" class="color-hex" />
            </div>
          </label>

          <label class="field">
            <span>Dot style</span>
            <select bind:value={dotType}>
              <option value="square">Square</option>
              <option value="dots">Dots</option>
              <option value="rounded">Rounded</option>
              <option value="classy">Classy</option>
              <option value="classy-rounded">Classy Rounded</option>
              <option value="extra-rounded">Extra Rounded</option>
            </select>
          </label>

          <label class="field">
            <span>Corner style</span>
            <select bind:value={cornerType}>
              <option value="square">Square</option>
              <option value="extra-rounded">Rounded</option>
              <option value="dot">Dot</option>
            </select>
          </label>

          <label class="field">
            <span>Size: {qrSize}px</span>
            <input type="range" min="200" max="600" step="50" bind:value={qrSize} />
          </label>
        </div>

        <!-- Logo upload -->
        <div class="logo-upload">
          <span>Logo / Icon (optional)</span>
          {#if logoFile}
            <div class="logo-preview-row">
              <img src={logoDataUrl} alt="logo preview" class="logo-thumb" />
              <span class="logo-name">{logoFile.name}</span>
              <button class="remove-logo" on:click={removeLogo}>✕ Remove</button>
            </div>
          {:else}
            <label class="upload-btn">
              📁 Upload image
              <input type="file" accept="image/*" on:change={handleLogoUpload} style="display:none" />
            </label>
          {/if}
        </div>
      </div>

    </section>

    <!-- RIGHT: Preview + Download -->
    <section class="preview-panel">
      <div class="preview-sticky">
        <h3>Preview</h3>
        <div class="qr-wrapper" bind:this={qrContainer}></div>

        <div class="download-btns">
          <button class="btn-primary" on:click={downloadPNG}>⬇ Download PNG</button>
          <button class="btn-secondary" on:click={downloadSVG}>⬇ Download SVG</button>
        </div>

        <p class="static-note">
          ✅ This is a <strong>static</strong> QR code.<br />
          It encodes your data directly — no redirect, no server, no expiry.
        </p>
      </div>
    </section>

  </div>
</main>

<!-- Explainer Section -->
<section class="explainer">
  <div class="explainer-inner">

    <div class="explainer-hero">
      <h2>Other sites give you a free QR code, then charge you to keep it working.<br/>Ours just works. Forever.</h2>
      <p>No subscriptions. No accounts. No surprises. Just scan it — today, next year, in ten years.</p>
    </div>

    <div class="explainer-grid">

      <div class="explainer-card">
        <div class="card-icon">🔗</div>
        <h3>Why do QR codes expire?</h3>
        <p>Most QR code generators create <strong>dynamic</strong> codes — the QR image doesn't link to your destination directly. Instead it links to <em>their</em> server, which redirects to your URL. When you stop paying, their server stops redirecting, and your QR code breaks.</p>
      </div>

      <div class="explainer-card">
        <div class="card-icon">✅</div>
        <h3>What makes ours different?</h3>
        <p>Every QR code generated here is <strong>static</strong> — your data is encoded directly into the image itself, in the pattern of black and white squares. There is no redirect, no middleman, no server involved. The code works as long as the image exists.</p>
      </div>

      <div class="explainer-card">
        <div class="card-icon">🔒</div>
        <h3>Your data never leaves your device</h3>
        <p>The QR code is generated entirely in your browser using JavaScript. Nothing you type is sent to any server. We have no database, no user accounts, and nothing to sell you. The site exists purely as a free utility.</p>
      </div>

      <div class="explainer-card">
        <div class="card-icon">📐</div>
        <h3>When should you use a dynamic QR code?</h3>
        <p>Dynamic codes make sense if you need to <strong>change the destination URL</strong> after printing (e.g. packaging printed at scale) or need <strong>scan analytics</strong>. For everything else — your website, WiFi, business card, event page — a static code is simpler, free, and more reliable.</p>
      </div>

    </div>

    <div class="faq">
      <h2>Frequently asked questions</h2>

      <details>
        <summary>Will my QR code really never expire?</summary>
        <p>Yes. A static QR code is just an image encoding your data. As long as you keep the image, it works. ForeverFreeQRCode.org doesn't host or serve your QR code — you download it and own it completely.</p>
      </details>

      <details>
        <summary>Do I need to create an account?</summary>
        <p>No. There are no accounts, no sign-ups, no email required. Open the page, generate your code, download it.</p>
      </details>

      <details>
        <summary>Why did my QR code from another site expire?</summary>
        <p>It was a dynamic QR code. The image encoded a short URL on that company's server (e.g. <em>qr.example.com/abc123</em>), which redirected to your real URL. When the company's free tier ended or you stopped paying, that redirect stopped working — even though your actual destination (your website, your page) is perfectly fine.</p>
      </details>

      <details>
        <summary>Can I put a logo on my QR code?</summary>
        <p>Yes — use the "Logo / Icon" upload in the Customize section. The QR code automatically uses a higher error correction level when a logo is added, so it remains scannable even with part of the pattern covered.</p>
      </details>

      <details>
        <summary>What file formats can I download?</summary>
        <p>PNG (best for most uses — emails, documents, web) and SVG (vector format, scales to any size without quality loss — ideal for print).</p>
      </details>

      <details>
        <summary>Is there a limit on how many QR codes I can generate?</summary>
        <p>No limit at all. Generate as many as you need.</p>
      </details>
    </div>

  </div>
</section>

<!-- Footer -->
<footer>
  <p>
    <strong>ForeverFreeQRCode.org</strong> — All QR codes are generated in your browser.
    No data is sent to any server. No tracking. Free forever.
  </p>
</footer>

<style>
  /* ── Layout ── */
  header {
    background: #1e293b;
    color: white;
    padding: 1rem 1.5rem;
  }
  .header-inner {
    max-width: 1100px;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
  }
  .logo {
    display: flex;
    align-items: center;
    gap: 0.6rem;
  }
  .logo-icon {
    width: 2rem;
    height: 2rem;
    border-radius: 6px;
    flex-shrink: 0;
  }
  .logo-title {
    font-size: 1.4rem;
    font-weight: 700;
    letter-spacing: -0.5px;
  }
  .accent { color: #38bdf8; }
  .logo-sub { color: #94a3b8; font-size: 1rem; }
  .tagline {
    font-size: 0.85rem;
    color: #94a3b8;
    margin-left: 2.4rem;
  }

  main {
    max-width: 1100px;
    margin: 1.5rem auto;
    padding: 0 1rem;
  }

  .app-grid {
    display: grid;
    grid-template-columns: 1fr 320px;
    gap: 1.5rem;
    align-items: start;
  }

  /* ── Form panel ── */
  .form-panel {
    background: white;
    border-radius: 12px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.08);
    overflow: hidden;
  }

  /* ── Tabs ── */
  .tabs {
    display: flex;
    flex-wrap: wrap;
    background: #f1f5f9;
    border-bottom: 1px solid #e2e8f0;
    padding: 0.5rem 0.5rem 0;
    gap: 0.25rem;
  }
  .tab-btn {
    display: flex;
    align-items: center;
    gap: 0.3rem;
    padding: 0.45rem 0.75rem;
    border: none;
    background: transparent;
    border-radius: 6px 6px 0 0;
    cursor: pointer;
    font-size: 0.85rem;
    color: #64748b;
    white-space: nowrap;
    transition: background 0.15s, color 0.15s;
  }
  .tab-btn:hover { background: #e2e8f0; color: #1e293b; }
  .tab-btn.active {
    background: white;
    color: #0ea5e9;
    font-weight: 600;
    box-shadow: 0 -1px 0 #0ea5e9 inset;
  }
  .tab-icon { font-size: 1rem; }
  .tab-label { }

  /* ── Tab content ── */
  .tab-content {
    padding: 1.25rem;
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
  }

  .field {
    display: flex;
    flex-direction: column;
    gap: 0.3rem;
    font-size: 0.85rem;
    color: #475569;
    font-weight: 500;
  }
  .field input, .field select, .field textarea {
    padding: 0.5rem 0.65rem;
    border: 1px solid #cbd5e1;
    border-radius: 6px;
    font-size: 0.9rem;
    color: #1e293b;
    background: #f8fafc;
    transition: border-color 0.15s;
  }
  .field input:focus, .field select:focus, .field textarea:focus {
    outline: none;
    border-color: #38bdf8;
    background: white;
  }
  .field.checkbox {
    flex-direction: row;
    align-items: center;
    gap: 0.5rem;
  }
  .field-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 0.75rem;
  }
  .hint {
    font-size: 0.8rem;
    color: #f59e0b;
    background: #fffbeb;
    padding: 0.4rem 0.6rem;
    border-radius: 6px;
    border: 1px solid #fde68a;
  }
  .warn-hint {
    color: #b45309;
  }
  .char-count {
    float: right;
    font-weight: 400;
    font-size: 0.78rem;
    color: #94a3b8;
  }
  .char-count.warn { color: #f59e0b; }
  .char-count.error { color: #ef4444; font-weight: 600; }

  /* ── Customization ── */
  .customization {
    padding: 1.25rem;
    border-top: 1px solid #f1f5f9;
  }
  .customization h3 {
    font-size: 0.9rem;
    color: #94a3b8;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    margin-bottom: 0.75rem;
  }
  .custom-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 0.75rem;
  }
  .color-row {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }
  .color-row input[type="color"] {
    width: 36px;
    height: 34px;
    padding: 2px;
    border: 1px solid #cbd5e1;
    border-radius: 6px;
    cursor: pointer;
    background: white;
  }
  .color-hex {
    flex: 1;
    font-family: monospace;
  }

  /* Logo upload */
  .logo-upload {
    margin-top: 0.75rem;
    display: flex;
    flex-direction: column;
    gap: 0.4rem;
    font-size: 0.85rem;
    color: #475569;
    font-weight: 500;
  }
  .upload-btn {
    display: inline-flex;
    align-items: center;
    gap: 0.4rem;
    padding: 0.45rem 0.85rem;
    background: #f1f5f9;
    border: 1px dashed #cbd5e1;
    border-radius: 6px;
    cursor: pointer;
    font-size: 0.85rem;
    color: #475569;
    transition: background 0.15s;
    width: fit-content;
  }
  .upload-btn:hover { background: #e2e8f0; }
  .logo-preview-row {
    display: flex;
    align-items: center;
    gap: 0.6rem;
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 6px;
    padding: 0.4rem 0.65rem;
  }
  .logo-thumb {
    width: 32px;
    height: 32px;
    object-fit: contain;
    border-radius: 4px;
  }
  .logo-name {
    flex: 1;
    font-size: 0.8rem;
    color: #64748b;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  .remove-logo {
    border: none;
    background: none;
    color: #ef4444;
    cursor: pointer;
    font-size: 0.8rem;
    padding: 0.2rem 0.4rem;
    border-radius: 4px;
  }
  .remove-logo:hover { background: #fef2f2; }

  /* ── Preview panel ── */
  .preview-panel {
    background: white;
    border-radius: 12px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.08);
  }
  .preview-sticky {
    position: sticky;
    top: 1rem;
    padding: 1.25rem;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1rem;
  }
  .preview-sticky h3 {
    font-size: 0.9rem;
    color: #94a3b8;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    align-self: flex-start;
  }
  .qr-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 200px;
  }
  .download-btns {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    width: 100%;
  }
  .btn-primary, .btn-secondary {
    padding: 0.65rem 1rem;
    border: none;
    border-radius: 8px;
    font-size: 0.9rem;
    font-weight: 600;
    cursor: pointer;
    transition: opacity 0.15s;
    width: 100%;
  }
  .btn-primary:hover, .btn-secondary:hover { opacity: 0.85; }
  .btn-primary {
    background: #0ea5e9;
    color: white;
  }
  .btn-secondary {
    background: #f1f5f9;
    color: #1e293b;
    border: 1px solid #e2e8f0;
  }
  .static-note {
    font-size: 0.78rem;
    color: #64748b;
    text-align: center;
    background: #f0fdf4;
    border: 1px solid #bbf7d0;
    border-radius: 8px;
    padding: 0.6rem 0.75rem;
    line-height: 1.5;
  }

  /* ── Explainer ── */
  .explainer {
    background: white;
    border-top: 1px solid #e2e8f0;
    padding: 3rem 1rem;
  }
  .explainer-inner {
    max-width: 1100px;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    gap: 3rem;
  }
  .explainer-hero {
    text-align: center;
    max-width: 780px;
    margin: 0 auto;
  }
  .explainer-hero h2 {
    font-size: 1.6rem;
    font-weight: 700;
    color: #1e293b;
    line-height: 1.35;
    margin-bottom: 0.75rem;
  }
  .explainer-hero p {
    font-size: 1.05rem;
    color: #64748b;
  }
  .explainer-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 1.25rem;
  }
  .explainer-card {
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 12px;
    padding: 1.5rem;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }
  .card-icon {
    font-size: 1.8rem;
  }
  .explainer-card h3 {
    font-size: 1rem;
    font-weight: 700;
    color: #1e293b;
  }
  .explainer-card p {
    font-size: 0.9rem;
    color: #475569;
    line-height: 1.6;
  }
  .faq {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
  }
  .faq h2 {
    font-size: 1.3rem;
    font-weight: 700;
    color: #1e293b;
    margin-bottom: 0.25rem;
  }
  details {
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    overflow: hidden;
  }
  details[open] {
    border-color: #bae6fd;
  }
  summary {
    padding: 1rem 1.25rem;
    font-size: 0.95rem;
    font-weight: 600;
    color: #1e293b;
    cursor: pointer;
    list-style: none;
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: #f8fafc;
  }
  summary::-webkit-details-marker { display: none; }
  summary::after {
    content: '+';
    font-size: 1.2rem;
    color: #0ea5e9;
    font-weight: 400;
  }
  details[open] summary::after {
    content: '−';
  }
  details p {
    padding: 1rem 1.25rem;
    font-size: 0.9rem;
    color: #475569;
    line-height: 1.7;
    background: white;
    border-top: 1px solid #e2e8f0;
  }

  @media (max-width: 768px) {
    .explainer-grid {
      grid-template-columns: 1fr;
    }
    .explainer-hero h2 {
      font-size: 1.25rem;
    }
  }

  /* ── Footer ── */
  footer {
    text-align: center;
    padding: 1.5rem 1rem;
    font-size: 0.8rem;
    color: #94a3b8;
    border-top: 1px solid #e2e8f0;
    margin-top: 2rem;
  }

  /* ── Mobile ── */
  @media (max-width: 768px) {
    .app-grid {
      grid-template-columns: 1fr;
    }
    .preview-sticky {
      position: static;
    }
    .custom-grid {
      grid-template-columns: 1fr;
    }
    .field-row {
      grid-template-columns: 1fr;
    }
    .tabs {
      gap: 0.15rem;
    }
    .tab-btn {
      padding: 0.4rem 0.6rem;
      font-size: 0.8rem;
    }
    .tab-label {
      display: none;
    }
    .tab-icon {
      font-size: 1.2rem;
    }
  }
</style>
