<svelte:options tag="crypto-wallet-widget" />

<script lang="ts">
  import { AwesomeQR, Options as QRCodeOptions } from "awesome-qr";

  let dataURL: string;

  /* Props */
  export let currency: string | undefined;
  export let walletAddress: string = $$props["wallet-address"];
  export let walletAddressColor: string =
    $$props["wallet-address-color"] ?? $$props["wallet-address-colour"] ?? "currentColor";
  export let icon: string | undefined;
  /* QR-Code */
  export let qrCodeSize: number = $$props["qr-code-size"] ?? 300;
  export let qrCodeDotColor: string | undefined = $$props["qr-code-dot-color"];
  export let qrCodeDotScale: number = $$props["qr-code-dot-scale"] ?? 1;
  export let qrCodeBackgroundColor: string | undefined = $$props["qr-code-background-color"];
  export let qrCodeLogoImage: string | undefined = $$props["qr-code-logo-image"];
  export let qrCodeLogoScale: number | undefined = $$props["qr-code-logo-scale"] ?? 0.3;
  export let qrCodeLogoMargin: number | undefined = $$props["qr-code-logo-margin"] ?? 5;
  export let qrCodeLogoCornerRadius: number | undefined =
    $$props["qr-code-logo-corner-radius"] ?? 40;
  $: getQRCodeDataURL(walletAddress).then((url) => (dataURL = url));

  async function getDataURLFromImage(url: string): Promise<string> {
    const blob = await fetch(url).then((r) => r.blob());
    return await new Promise((resolve) => {
      let reader = new FileReader();
      reader.onload = () => resolve(reader.result.toString());
      reader.readAsDataURL(blob);
    });
  }

  async function getQRCodeDataURL(text: string): Promise<string> {
    if (!text) return;

    const options: Partial<QRCodeOptions> = {
      components: {
        data: {
          scale: qrCodeDotScale,
        },
        timing: {
          scale: qrCodeDotScale + 0.1,
          protectors: false,
        },
        alignment: {
          scale: qrCodeDotScale + 0.1,
          protectors: false,
        },
        cornerAlignment: {
          scale: qrCodeDotScale + 0.1,
          protectors: true,
        },
      },
    };

    options.text = text;

    if (qrCodeSize) options.size = qrCodeSize;
    if (qrCodeDotColor) options.colorDark = qrCodeDotColor;
    if (qrCodeBackgroundColor) options.colorLight = qrCodeBackgroundColor;
    if (qrCodeLogoImage) {
      try {
        const dataURL = await getDataURLFromImage(qrCodeLogoImage);
        options.logoImage = dataURL;
      } catch (e) {
        console.error(e);
      }
    }
    if (qrCodeLogoScale) options.logoScale = qrCodeLogoScale;
    if (qrCodeLogoMargin) options.logoMargin = qrCodeLogoMargin;
    if (qrCodeLogoCornerRadius) options.logoCornerRadius = qrCodeLogoCornerRadius;

    return new Promise((resolve) => {
      new AwesomeQR(options).draw().then((dataURL) => {
        resolve(dataURL.toString());
      });
    });
  }

  let popupActive = false;
  let showCopied = false;

  async function copyWalletAddressToClipboard() {
    await navigator.clipboard.writeText(walletAddress);
    showCopied = true;
    setTimeout(() => {
      showCopied = false;
    }, 3000);
  }
</script>

<div class="crypto-wallet-widget">
  {#if walletAddress}
    <div class="widget-content">
      {#if icon}
        <img src={icon} alt="Crypto wallet widget by Dan6erbond" class="icon-img flex-shrink-0" />
      {/if}
      <div class="main-txt">
        {#if currency}
          <span class="currency">{currency}</span>
        {/if}
        <span class="wallet-address" style={`color: ${walletAddressColor}`}>{walletAddress}</span>
      </div>
      <div class="flex-grow" />
      <button class="copy-btn flex-shrink-0" on:click={copyWalletAddressToClipboard}>
        <span class="copied-txt" class:active={showCopied}>Copied!</span>
        <svg
          xmlns="http://www.w3.org/2000/svg"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5a2 2 0 002 2h2a2 2 0 002-2M8 5a2 2 0 012-2h2a2 2 0 012 2m0 0h2a2 2 0 012 2v3m2 4H10m0 0l3-3m-3 3l3 3"
          />
        </svg>
      </button>
      <button class="qrcode-btn flex-shrink-0" on:click={() => (popupActive = true)}>
        <svg
          xmlns="http://www.w3.org/2000/svg"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm12 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z"
          />
        </svg>
      </button>
    </div>
    <div class="popup" class:active={popupActive} on:click={() => (popupActive = false)}>
      <img src={dataURL} class="qrcode-preview" alt="Crypto wallet widget by Dan6erbond" />
    </div>
  {/if}
</div>

<style>
  .crypto-wallet-widget .widget-content {
    display: flex;
    align-items: center;
    flex-wrap: nowrap;
  }

  .icon-img {
    height: 2rem;
    width: 2rem;
    margin-right: 0.5rem;
  }

  .copy-btn,
  .qrcode-btn {
    cursor: pointer;
    padding: 0;
    border: none;
    background: none;
    margin-left: 0.5rem;
    color: currentColor;
    display: flex;
    align-items: center;
  }

  .copy-btn svg,
  .qrcode-btn svg {
    height: 1.75rem;
    width: 1.75rem;
  }

  .copied-txt {
    display: none;
    margin-right: 0.5rem;
  }

  .copied-txt.active {
    display: block;
  }

  .popup {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: rgba(0, 0, 0, 0.5);
    z-index: 1;
    visibility: hidden;
    opacity: 0;
    transition: opacity 0.25s linear;
  }

  .popup.active {
    visibility: visible;
    opacity: 1;
    transition: visibility 0.25s linear, opacity 0.25s linear;
  }

  .flex-grow {
    flex-grow: 1;
  }

  .flex-1 {
    flex: 1;
  }

  .flex-shrink-0 {
    flex-shrink: 0;
  }

  .main-txt {
    flex-shrink: 1;
    min-width: 0;
  }

  .main-txt .currency {
    font-size: 1.25rem;
    display: flex;
    flex-direction: column;
    min-width: 0;
  }

  .main-txt .wallet-address {
    overflow-wrap: break-word;
    text-overflow: ellipsis;
    min-width: 0;
  }
</style>
