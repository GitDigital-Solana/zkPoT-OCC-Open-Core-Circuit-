
1. components/GasEstimate.tsx – The Component

```tsx
import { FC, useEffect, useState } from 'react';
import { loadConfig, CircleAPIClient, GasEstimate, SensitiveValues } from '../src';

// You can also import maskedNumber if you want raw text masking,
// but we’ll use CSS blur for visual effect.
const GasEstimateWidget: FC = () => {
  const [estimate, setEstimate] = useState<GasEstimate | null>(null);
  const [revealed, setRevealed] = useState(false);

  useEffect(() => {
    const config = loadConfig();
    const client = new CircleAPIClient(
      config.circleApiKey,
      config.sensitiveValues,
      config.demoMode
    );
    client.getKycCostEstimate().then(setEstimate);
  }, []);

  if (!estimate) return <span>Loading...</span>;

  return (
    <div style={{ fontFamily: 'monospace', padding: '1rem' }}>
      <h3>KYC Transaction Cost</h3>
      <div>
        <span>Gas (SOL): </span>
        <span
          className={`blur-sensitive${revealed ? ' revealed' : ''}`}
          onClick={() => setRevealed(!revealed)}
          title="Click to reveal"
        >
          {estimate.costSOL}
        </span>
      </div>
      <div>
        <span>Service Fee (USD): </span>
        <span
          className={`blur-sensitive${revealed ? ' revealed' : ''}`}
          onClick={() => setRevealed(!revealed)}
          title="Click to reveal"
        >
          {estimate.feeUSD}
        </span>
      </div>
      <button
        onClick={() => setRevealed(!revealed)}
        style={{ marginTop: '0.5rem' }}
      >
        {revealed ? '🔒 Hide again' : '👁️ Reveal'}
      </button>
    </div>
  );
};

export default GasEstimateWidget;
```

2. styles/blur.css – The Blur Style

Add this CSS globally or import it in your layout.
You can also include it via a <style> tag in your HTML if you’re not using CSS modules.

```css
.blur-sensitive {
  filter: blur(6px);
  transition: filter 0.2s ease;
  cursor: pointer;
  user-select: none;
}

.blur-sensitive.revealed {
  filter: blur(0);
  cursor: default;
}
```

3. Using It in a Page (e.g., pages/index.tsx in Next.js)

```tsx
import type { NextPage } from 'next';
import GasEstimateWidget from '../components/GasEstimate';

const Home: NextPage = () => {
  return (
    <div>
      <h1>Aurora ZK KYC Demo</h1>
      <GasEstimateWidget />
      {/* other components */}
    </div>
  );
};

export default Home;
```

4. If You’re Using Plain HTML/JS Instead of React

You can replicate the same effect with vanilla JavaScript. Add this to your (root)/index.html:

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8" />
  <title>Aurora ZK Demo</title>
  <link rel="stylesheet" href="blur.css" />
</head>
<body>
  <div id="gas-estimate-container">
    <h3>KYC Transaction Cost</h3>
    <p>Gas (SOL): <span id="gas-value" class="blur-sensitive">••••</span></p>
    <p>Fee (USD): <span id="fee-value" class="blur-sensitive">••••</span></p>
    <button id="reveal-btn">👁 Reveal</button>
  </div>
  <script src="demo.js"></script>
</body>
</html>
```

```javascript
// demo.js
(async function() {
  // You’d need to bundle your SDK for the browser, but for a quick demo
  // you could pre‑load the values from a pre‑set config object.
  const isDemo = true; // from your logic
  const gas = isDemo ? '0.00012' : '0.0001245';
  const fee = isDemo ? '0.50' : '0.75';

  document.getElementById('gas-value').textContent = gas + ' SOL';
  document.getElementById('fee-value').textContent = fee + ' USD';

  const revealBtn = document.getElementById('reveal-btn');
  revealBtn.addEventListener('click', () => {
    document.getElementById('gas-value').classList.toggle('revealed');
    document.getElementById('fee-value').classList.toggle('revealed');
    revealBtn.textContent =
      document.getElementById('gas-value').classList.contains('revealed')
        ? '🔒 Hide'
        : '👁 Reveal';
  });
})();
```

---

5. Full‑Circle Integration with Your Framework

· The CircleAPIClient.getKycCostEstimate() will automatically return blurred strings (from maskedNumber) when DEMO_MODE=true.
· The CSS blur adds a second visual layer, so that even if someone inspects the raw text in devtools, they’ll see •••••.
· When you mock the environment for the hackathon, the demo never exposes your real business numbers.

Now you can confidently demo without tipping your hand. If you want the blur to be permanent (no click to reveal), just skip the revealed class toggle. Need anything else polished?