
**Unet**


<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<title>U-Net</title>
<style>
  body { font-family: -apple-system, "Noto Sans TC", "PingFang TC", sans-serif; background:#f7f7f5; margin:0; padding:20px; }
  .wrap { max-width: 1000px; margin: 0 auto; }
  h2 { text-align:center; color:#222; margin-bottom:4px; }
  .sub { text-align:center; color:#666; font-size:13px; margin-bottom:16px; }
  svg { width:100%; height:auto; display:block; background:#fff; border-radius:8px; box-shadow:0 1px 4px rgba(0,0,0,0.08); }
  text { font-family: -apple-system, "Noto Sans TC", "PingFang TC", sans-serif; }
</style>
</head>
<body>
<div class="wrap">
<svg viewBox="0 0 1000 780" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrowBlue" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto">
      <path d="M0,0 L8,4 L0,8 Z" fill="#2b6cb0"/>
    </marker>
    <marker id="arrowRed" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto">
      <path d="M0,0 L8,4 L0,8 Z" fill="#c53030"/>
    </marker>
    <marker id="arrowGreen" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto">
      <path d="M0,0 L8,4 L0,8 Z" fill="#2f855a"/>
    </marker>
  </defs>

  <!-- ===== Encoder boxes ===== -->
  <!-- Level 1: 64ch -->
  <rect x="40" y="30" width="200" height="70" rx="6" fill="#ebf4ff" stroke="#2b6cb0" stroke-width="1.5"/>
  <text x="140" y="50" text-anchor="middle" font-size="13" fill="#1a365d" font-weight="600">64 channels</text>
  <text x="140" y="68" text-anchor="middle" font-size="12" fill="#2d3748">Conv3×3 ×2 + ReLU</text>
  <text x="140" y="84" text-anchor="middle" font-size="12" fill="#4a5568">H × W</text>

  <!-- Level 2: 128ch -->
  <rect x="40" y="170" width="200" height="70" rx="6" fill="#ebf4ff" stroke="#2b6cb0" stroke-width="1.5"/>
  <text x="140" y="190" text-anchor="middle" font-size="13" fill="#1a365d" font-weight="600">128 channels</text>
  <text x="140" y="208" text-anchor="middle" font-size="12" fill="#2d3748">Conv3×3 ×2 + ReLU</text>
  <text x="140" y="224" text-anchor="middle" font-size="12" fill="#4a5568">H/2 × W/2</text>

  <!-- Level 3: 256ch -->
  <rect x="40" y="310" width="200" height="70" rx="6" fill="#ebf4ff" stroke="#2b6cb0" stroke-width="1.5"/>
  <text x="140" y="330" text-anchor="middle" font-size="13" fill="#1a365d" font-weight="600">256 channels</text>
  <text x="140" y="348" text-anchor="middle" font-size="12" fill="#2d3748">Conv3×3 ×2 + ReLU</text>
  <text x="140" y="364" text-anchor="middle" font-size="12" fill="#4a5568">H/4 × W/4</text>

  <!-- Level 4: 512ch -->
  <rect x="40" y="450" width="200" height="70" rx="6" fill="#ebf4ff" stroke="#2b6cb0" stroke-width="1.5"/>
  <text x="140" y="470" text-anchor="middle" font-size="13" fill="#1a365d" font-weight="600">512 channels</text>
  <text x="140" y="488" text-anchor="middle" font-size="12" fill="#2d3748">Conv3×3 ×2 + ReLU</text>
  <text x="140" y="504" text-anchor="middle" font-size="12" fill="#4a5568">H/8 × W/8</text>

  <!-- Downsample arrows + labels (encoder) -->
  <line x1="140" y1="100" x2="140" y2="168" stroke="#2b6cb0" stroke-width="2" marker-end="url(#arrowBlue)"/>
  <text x="150" y="138" font-size="11" fill="#2b6cb0">MaxPool 2×2</text>

  <line x1="140" y1="240" x2="140" y2="308" stroke="#2b6cb0" stroke-width="2" marker-end="url(#arrowBlue)"/>
  <text x="150" y="278" font-size="11" fill="#2b6cb0">MaxPool 2×2</text>

  <line x1="140" y1="380" x2="140" y2="448" stroke="#2b6cb0" stroke-width="2" marker-end="url(#arrowBlue)"/>
  <text x="150" y="418" font-size="11" fill="#2b6cb0">MaxPool 2×2</text>

  <line x1="140" y1="520" x2="140" y2="588" stroke="#2b6cb0" stroke-width="2" marker-end="url(#arrowBlue)"/>
  <text x="150" y="558" font-size="11" fill="#2b6cb0">MaxPool 2×2</text>

  <!-- ===== Bottleneck ===== -->
  <rect x="40" y="590" width="200" height="70" rx="6" fill="#e6ffed" stroke="#2f855a" stroke-width="1.5"/>
  <text x="140" y="610" text-anchor="middle" font-size="13" fill="#22543d" font-weight="600">1024 channels (bottleneck)</text>
  <text x="140" y="628" text-anchor="middle" font-size="12" fill="#2d3748">Conv3×3 ×2 + ReLU</text>
  <text x="140" y="644" text-anchor="middle" font-size="12" fill="#4a5568">H/16 × W/16</text>

  <!-- bottleneck to decoder -->
  <path d="M240,625 C500,625 500,625 700,590" fill="none" stroke="#2f855a" stroke-width="2" marker-end="url(#arrowGreen)"/>

  <!-- ===== Decoder boxes ===== -->
  <!-- Level 4: 512ch -->
  <rect x="700" y="450" width="220" height="70" rx="6" fill="#fff5f5" stroke="#c53030" stroke-width="1.5"/>
  <text x="810" y="470" text-anchor="middle" font-size="13" fill="#742a2a" font-weight="600">512 channels</text>
  <text x="810" y="488" text-anchor="middle" font-size="12" fill="#2d3748">Concat + Conv3×3 ×2 + ReLU</text>
  <text x="810" y="504" text-anchor="middle" font-size="12" fill="#4a5568">H/8 × W/8</text>

  <!-- Level 3: 256ch -->
  <rect x="700" y="310" width="220" height="70" rx="6" fill="#fff5f5" stroke="#c53030" stroke-width="1.5"/>
  <text x="810" y="330" text-anchor="middle" font-size="13" fill="#742a2a" font-weight="600">256 channels</text>
  <text x="810" y="348" text-anchor="middle" font-size="12" fill="#2d3748">Concat + Conv3×3 ×2 + ReLU</text>
  <text x="810" y="364" text-anchor="middle" font-size="12" fill="#4a5568">H/4 × W/4</text>

  <!-- Level 2: 128ch -->
  <rect x="700" y="170" width="220" height="70" rx="6" fill="#fff5f5" stroke="#c53030" stroke-width="1.5"/>
  <text x="810" y="190" text-anchor="middle" font-size="13" fill="#742a2a" font-weight="600">128 channels</text>
  <text x="810" y="208" text-anchor="middle" font-size="12" fill="#2d3748">Concat + Conv3×3 ×2 + ReLU</text>
  <text x="810" y="224" text-anchor="middle" font-size="12" fill="#4a5568">H/2 × W/2</text>

  <!-- Level 1: 64ch (output) -->
  <rect x="700" y="30" width="220" height="70" rx="6" fill="#fff5f5" stroke="#c53030" stroke-width="1.5"/>
  <text x="810" y="50" text-anchor="middle" font-size="13" fill="#742a2a" font-weight="600">64 channels</text>
  <text x="810" y="68" text-anchor="middle" font-size="12" fill="#2d3748">Concat + Conv3×3 ×2 + ReLU</text>
  <text x="810" y="84" text-anchor="middle" font-size="12" fill="#4a5568">H × W</text>

  <!-- output head -->
  <rect x="700" y="-10" width="0" height="0"/>
  <line x1="810" y1="30" x2="810" y2="0" stroke="none"/>

  <!-- Upsample arrows (decoder, going upward) -->
  <line x1="810" y1="448" x2="810" y2="382" stroke="#c53030" stroke-width="2" marker-end="url(#arrowRed)"/>
  <text x="820" y="418" font-size="11" fill="#c53030">Transposed Conv 2×2</text>

  <line x1="810" y1="308" x2="810" y2="242" stroke="#c53030" stroke-width="2" marker-end="url(#arrowRed)"/>
  <text x="820" y="278" font-size="11" fill="#c53030">Transposed Conv 2×2</text>

  <line x1="810" y1="168" x2="810" y2="102" stroke="#c53030" stroke-width="2" marker-end="url(#arrowRed)"/>
  <text x="820" y="138" font-size="11" fill="#c53030">Transposed Conv 2×2</text>

  <!-- Skip connections (encoder -> decoder concat), same level -->
  <line x1="240" y1="65" x2="700" y2="65" stroke="#e53e3e" stroke-width="1.5" stroke-dasharray="5,4" marker-end="url(#arrowRed)"/>
  <text x="470" y="58" text-anchor="middle" font-size="11" fill="#e53e3e">skip connection</text>

  <line x1="240" y1="205" x2="700" y2="205" stroke="#e53e3e" stroke-width="1.5" stroke-dasharray="5,4" marker-end="url(#arrowRed)"/>
  <text x="470" y="198" text-anchor="middle" font-size="11" fill="#e53e3e">skip connection</text>

  <line x1="240" y1="345" x2="700" y2="345" stroke="#e53e3e" stroke-width="1.5" stroke-dasharray="5,4" marker-end="url(#arrowRed)"/>
  <text x="470" y="338" text-anchor="middle" font-size="11" fill="#e53e3e">skip connection</text>

  <line x1="240" y1="485" x2="700" y2="485" stroke="#e53e3e" stroke-width="1.5" stroke-dasharray="5,4" marker-end="url(#arrowRed)"/>
  <text x="470" y="478" text-anchor="middle" font-size="11" fill="#e53e3e">skip connection</text>

  <!-- Output arrow -->
  <line x1="920" y1="65" x2="970" y2="65" stroke="#333" stroke-width="2" marker-end="url(#arrowBlue)"/>
  <text x="975" y="70" font-size="11" fill="#333">1×1 Conv → output</text>



  <text x="500" y="20" text-anchor="middle" font-size="13" fill="#555" font-weight="600">Encoder（Down Sampling）</text>
  <text x="810" y="20" text-anchor="middle" font-size="13" fill="#555" font-weight="600" transform="translate(0,0)"></text>
</svg>

</div>
</body>
</html>
- **Contracting Path/ Encoder**: Two $3*3\text{Conv}$ + $\text{ReLU}$ 
	: each layer Down sampling will let Half image resolution, Double Channel($64\rightarrow 128\rightarrow 256\rightarrow 512 \rightarrow 1024$)

- **Bottleneck**: same convolution, give global context to decoder not skip 

- **Expansive Path/ Decoder**: Two $2*2\text{up-Conv}+\text{channel-wise concatenation}$
	:each layer Up sampling will let Double image resolution, Half Channel
	$\text{channel-wise concatenation}:$ 
		$x^i_{decoder}=Concat(Upsample(x^{i-1}_{decoder}), x^i_{encoder})$ 

- **Skip - Connection**
	:transport encoder output to correspond decoder layer, Restore spatial detail information loss on downsampling



- Standard CNN problem
	:On Inpainting Tesk(image restoration or replacement) will identically treat other invalid pixel(0 or Nan .. etc value), cause color discrepancy and blurriness


**PConv(Partial Convolution)**
:only output valid value

$x' = \begin{cases}W^T(X \odot M)\frac{sum(\mathbf{1})}{sum{M}}+b,& \text{if sum(M)>0}\\0,& \text{otherwise}\end{cases}$  

note
	$\frac{sum(\mathbf{1})}{sum{M}}$ : used to renormalization, because when$\text{num of pixel} \downarrow$ (masked)will let $\text{size} \downarrow$ 
mask - update: mask will update before each partial convolution
	$m'=\begin{cases}1,& if sum(M)>0 \\ 0, & if sum(M)=0\end{cases}$ 
	so if exist pixel, next layer will be valid

**GConv(Gated Convolution)**
: let model learn **Dynamic mask** $\in [0,1]$ 
	$y = \phi(W_f * X) \odot \sigma(W_g * X)$
		$\phi$:non-linear activation function(common way : LeakyReLU [[ReLU]], output: tanh)
		$W_f$: generate feature
		$W_g$: generate gating signal
		