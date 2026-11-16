# AdireTrust: Preserving Cultural Heritage & Artisan Livelihoods with Computer Vision

**An AI-powered authentication and digital provenance ecosystem designed to distinguish authentic hand-dyed Abeokuta Adire from industrial factory prints, protecting local artisans and restoring trust in African textiles.**

---

## Executive Summary

**AdireTrust** is an end-to-end computer vision and digital provenance platform built for the **Ogun Digital Summit Hackathon** (showcase at **Ogun Tech Hub, Kobape, Abeokuta**). 

The platform addresses a critical socio-economic crisis facing Ogun State: the mass influx of cheap, imported factory-printed textile imitations marketed as genuine, handmade Yoruba **Adire**. By analyzing microscopic dye-resist dynamics (bleed, crackle, soak-through, and repeat irregularities) using state-of-the-art vision models (DINOv2 / SigLIP), AdireTrust gives buyers, tourists, and traders an instant verification tool via **WhatsApp** or web, backed by a **QR Adire Passport** linking genuine pieces directly to the artisan who crafted them.

```
       [ Cloth Photo (Front + Back + Macro) ]
                          │
                          ▼
            ┌───────────────────────────┐
            │   AdireTrust Vision Core  │
            │  (DINOv2 / SigLIP + CAM)  │
            └─────────────┬─────────────┘
                          │
          ┌───────────────┴───────────────┐
          ▼                               ▼
 [ Authenticity Score ]          [ Attention Heatmap ]
   - Genuine Hand-Dyed             - Resist crackles
   - Factory Print                 - Edge bleed / soak
   - Abstain ("Flip / Re-shoot")   - Repeat regularity
          │
          ▼
 [ QR Adire Passport Generator ] ──▶ Artisan Attribution & Provenance
```

---

## The Problem: Why AdireTrust Matters

Abeokuta, Ogun State, is the historic cradle of Adire. Production centers like **Itoku Market** and **Kemta Adire Market** (near Olumo Rock) have sustained generations of master dyers—primarily women artisans—for over a century.

However, the local craft is facing an unprecedented existential threat:

* **Massive Market Displacement:** According to reporting by *The Punch*, industry stakeholders estimate that up to **80% of adire** sold across major Nigerian markets are cheap, industrial machine prints imported from abroad, causing **50–70 local women and youths** in production and marketing to lose their livelihoods every three weeks.
* **Severe Price Undercutting:** *AFP / France24* documented imported machine-printed imitations retailing at **₦3,300** (roughly half the price of genuine handmade cloth), severely undercutting Abeokuta's ~2,000 active merchants and producers.
* **Invisible Counterfeiting:** The *Nigerian Tribune* reported that the **Alake of Egbaland, Oba Adedotun Gbadebo**, established a dedicated committee in Itoku and Asero to address counterfeit adire, with traditional leadership emphasizing that ordinary buyers simply cannot distinguish sophisticated prints from genuine resist-dyed cloth.
* **Cultural Erosion:** *Daily Trust* traces the living heritage of Kemta directly to historic compounds like Jojolola's compound, supporting over 1,500 women dyers. Machine imitations strip away the cultural identity, traditional knowledge, and financial resilience of these communities.

AdireTrust was conceived to place the discerning eye of a veteran Abeokuta textile trader into any smartphone camera.

---

## How It Works: Technical Architecture & Visual Cues

Rather than analyzing broad garment silhouettes, AdireTrust inspects the **physical process markers** that distinguish manual resist dyeing (*Adire Oniko*, *Adire Eleko*, and *Batik/Kampala*) from flat industrial roller and digital screen printing:

### Key Microscopic & Structural Cues
1. **Dye Bleed & Capillary Diffusion:** In hand-dyed cloth, liquid dye wicks naturally into cotton fibers at resist boundaries, producing subtle gradient bleed. Factory prints feature razor-sharp, uniform pixel or halftone dot boundaries.
2. **Resist Lines & Crackle Dynamics:** Hand-applied cassava starch (*Eleko*) or paraffin/beeswax cracks organically during dipping, producing random, branching capillary veins. Factory prints either lack veins completely or repeat identical printed crackle patterns across pattern repeats.
3. **Double-Sided Penetration (Soak-Through):** Real hand-dipped dye penetrates through the cotton substrate, making motifs clearly legible on both the front and reverse faces. Industrial roller prints often deposit pigment solely on the front face, leaving a pale, washed-out back.
4. **Repeat Drift & Irregularity:** Hand block-stamping (*Adire Alabere/Batik Cap*) or manual freehand drawing drifts slightly in spacing and pressure across yards. Machine-printed cylinders produce mathematically identical repeats.

---

## Model & Transfer Learning Strategy

Because no public, labeled ground-truth dataset exists specifically comparing Abeokuta hand-dyed adire against Chinese factory counterfeits, we deploy a **staged transfer learning and feature probing pipeline**:

| Stage | Backbone / Data | Purpose |
|---|---|---|
| **Stage 1: General Foundation** | Frozen **DINOv2** (`ViT-S/14`, `ViT-B/14`) & **SigLIP** | Self-supervised visual backbones inherently retain low-level surface texture, grain, and high-frequency edge features without requiring labeled fine-tuning from scratch. |
| **Stage 2: Intermediate Transfer** | Indonesian Batik Dataset (BatikSnap), DTD (Describable Textures), AITEX, Wax-MNIST | Train on closest published physical analogs: *batik tulis/cap* (handcrafted) vs *batik printing* (machine print), teaching the network crackle and bleed physics. |
| **Stage 3: Curated Web & Negatives** | Wikimedia Commons, Museum Archives (V&A, British Museum), Verified Negatives (Ankara factory prints) | Seed initial prototype validation; perceptual hash (`phash`) deduplication ensures no duplicate photos bleed between splits. |
| **Stage 4: Ground-Truth Field Data** | Primary Abeokuta Market Collection (Itoku & Kemta markets) | Real-world benchmark: multi-phone, varied natural shade/sun lighting, front + back + macro crops, labeled with trader/artisan consensus. |

### Evaluation Principles
* **Prioritize Precision on Genuine:** In an artisan market, a false "genuine" issued for a cheap synthetic copy erodes consumer trust; an uncertain cloth is abstained rather than guessed.
* **Piece-Level Split Integrity:** All crops, fronts, backs, and macro shots of a single piece ID (e.g., `ITK-001`) remain strictly in the same split. Val/test splits also hold out an entire stall or market cluster.
* **Explainability:** Grad-CAM / Attention rollout heatmaps highlight *why* a classification was made (e.g., pointing directly to edge soak-through or crackle).

---

## System Modules & Deliverables

```
ogun-hackathon/
├── data/
│   ├── images/               # Raw and processed cloth captures
│   └── labels.csv            # Ground-truth annotations & metadata schema
├── notebooks/
│   ├── 01_scraping.ipynb     # Web ingestion, Commons downloader & pHash dedup
│   ├── 02_feature_extract.ipynb  # DINOv2 / SigLIP embedding pipeline
│   └── 03_evaluation.ipynb   # Precision-recall, ROC-AUC & ablation studies
├── backend/
│   ├── main.py               # FastAPI service: POST /check, GET /passport/{id}
│   ├── models/               # Model weights & inference runner
│   └── static/               # QR passport cards & heatmaps
├── requirements.txt          # Python dependencies
└── README.md                 # Project documentation
```

### The End-to-End User Experience

1. **WhatsApp Verification Bot:**
   * Powered by Twilio Sandbox / Meta WhatsApp Cloud API.
   * A market buyer snaps a close-up and reverse-side photo of a fabric.
   * The bot responds in seconds with:
     * Classification: **Genuine Hand-Dyed**, **Factory Print**, or **Abstain ("Please photograph the reverse side or get closer")**.
     * Confidence score.
     * Visual attention heatmap overlay showing where bleed or crackle was identified.
2. **QR Adire Passport:**
   * Consented master artisans receive a unique QR identifier for their batch.
   * Scanning the passport presents the buyer with a verified digital certificate containing the maker's workshop, craft method (e.g., *Eleko*, *Oniko*), date, and Abeokuta market origin.
3. **Offline Laptop UI:**
   * Standalone fallback web interface ensuring 100% demo uptime at the showcase booth without relying on venue connectivity.

---

## Data Annotation Schema (`data/labels.csv`)

Every collected image follows a strict attribution and metadata schema:

| Column | Description |
|---|---|
| `image_id` | Unique ID (e.g., `img_00001`, `ITK_001_F`) |
| `filename` | File name in `data/images/` |
| `source_url` | Origin web page or field camera ID |
| `source_page` | Contextual page reference or market location |
| `source_group` | Source grouping (`commons`, `field_itoku`, `field_kemta`, `museum`, `print_negative`) |
| `seller_label` | Original vendor or listing claim |
| `license_or_tos_note` | Creative Commons, open access, or written artisan consent identifier |
| `phash` | Perceptual image hash to detect and discard duplicates across datasets |
| `label` | Verified target: `genuine_hand_dyed`, `factory_print`, or `unusable` |
| `verified` | `yes` (confirmed by artisan/expert) / `no` |
| `notes` | Specific cloth details (front vs back, macro, dye type) |

---

## Getting Started

### 1. Clone & Set Up Environment

```bash
# Clone the repository
git clone https://github.com/<your-username>/AdireTrust.git
cd AdireTrust

# Create and activate virtual environment
python -m venv venv
.\venv\Scripts\activate   # Windows

# Install dependencies
pip install -r requirements.txt
```

### 2. Run Data Ingestion & Deduplication

Open and run `scraping.ipynb` to ingest open-access datasets, compute perceptual hashes, and verify licenses:

```bash
jupyter notebook scraping.ipynb
```
---

## 10-Day Hackathon Roadmap

```
Nov 15-16:  [Repo, Ingestion & Deduplication Pipeline] ──▶ (Complete)
Nov 17-18:  [Field Collection Days: Itoku & Kemta Markets]
Nov 19-20:  [DINOv2/SigLIP Feature Probes & Explainability Heatmaps]
Nov 21-22:  [FastAPI Backend, WhatsApp Bot & QR Passport Generator]
Nov 23-24:  [End-to-End Rehearsal, Offline Booth Fallback & Slide Deck]
Nov 25:     [Final Showcase at Ogun Tech Hub, Kobape, Abeokuta]
```

---

## Ethical Guidelines & Responsible AI

* **Informed Artisan Consent:** Field data collection is conducted using clear, bilingual (English & Yoruba) consent protocols. No personal phone numbers or unconsented portrait photos are retained.
* **No Speculative Scraping:** Web collections respect `robots.txt`, open licenses (CC BY, CC0), and academic fair-use standards.
* **Transparent Limitations:** Counterfeit textile detection via standard mobile cameras is a hard computer vision problem. Our model is built to **abstain** gracefully rather than hallucinate false confidence on noisy, ambiguous photos.
* **Artisan First:** The ultimate beneficiary of this tool is the Abeokuta dyer and the preservation of indigenous Yoruba artistic heritage.

---

## 👥 Team & Roles

* **Timi** — AI / Computer Vision Lead (Foundation model probing, feature extraction, evaluation metrics & explainability).
* **Teammate** — Field Operations, Labeling Infrastructure & Backend / WhatsApp / QR Demo Path.

---

*“Adire is not just cloth; it is the living script and pride of Egbaland. AdireTrust makes technology its guardian.”*
