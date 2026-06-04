
## KZN Provincial Financial Recovery Plan Platform

## Multi-Channel Communication Platform

> Province-wide government financial recovery plan dissemination system. Combines a React-based internal information portal with a cross-platform desktop notification application distributed to all department workstations.

---

## System Overview

The KZN Provincial Financial Recovery Plan Platform is a **multi-channel communication system** built for KZN Provincial Treasury. It solves the problem of ensuring every employee stays updated on the R12.4B+ provincial financial recovery strategy — without relying on email readership or browser bookmarks.

**Two integrated components:**

| Component | Purpose | Deployment |
|---|---|---|
| **React Information Portal** | Public-facing website with headless CMS, animated data visualizations, and role-based content | IIS / Web |
| **Desktop Notification App** | Cross-platform Tkinter application triggering 3x daily popups with direct portal links | Windows Workstations |

---

## Business Problem

The KZN Treasury needed to communicate the Provincial Financial Recovery Plan (2025–2030) to **every employee across all departments**. Existing methods failed:

- **Email fatigue** — Mass emails ignored; no guarantee of readership
- **No central source** — Updates scattered across PDFs, intranet pages, and meetings
- **Non-technical stakeholders** — Content managers couldn't update web content without developer help
- **No engagement tracking** — Leadership had no visibility into who was informed
- **Browser dependency** — Employees didn't bookmark internal sites

---

## Solution Architecture

### What We Built

| System Layer | Engineering Decision |
|---|---|
| **Frontend Portal** | Multi-page React SPA with scroll-triggered animations, expandable cards, timeline visualizations |
| **CMS Backend** | Headless JSON-driven content model — non-technical admins edit via password-protected dashboard |
| **Desktop Notifier** | Tkinter app compiled to native `.exe` via PyInstaller with optimized build pipeline |
| **Data Visualization** | Interactive bar charts, pressure indicators, unauthorized expenditure tracking |
| **Distribution** | Silent deployment across government workstations; triggers 3x daily with direct portal links |

---

## Engineering Highlights

### 🖥️ Desktop Notification System

Cross-platform Tkinter application compiled to native Windows executable via PyInstaller with custom `.spec` build pipeline.

**Build optimization:**
- Stripped 20+ unused modules
- Filtered TCL/TK bloat (demo files, unused widgets)
- Removed unnecessary encoding tables
- **Result:** Executable footprint minimized for silent workstation deployment

```python
# resize_assets.py — PIL-based asset preprocessing pipeline
# Run ONCE before building the exe. Eliminates runtime image processing dependencies.

from PIL import Image
import os

ASSETS = [
    ("treasury_logo.png",  "logo_header.png",  115, 42),
    ("treasury_logo.png",  "logo_bottom.png",  130, 58),
    ("NDP.png",            "ndp_bottom.png",   130, 58),
]

for src, dst, w, h in ASSETS:
    if not os.path.exists(src):
        print(f"  SKIP  {src} not found")
        continue
    img = Image.open(src).convert("RGBA")
    img.thumbnail((w, h), Image.LANCZOS)
    img.save(dst, "PNG")
    print(f"  OK    {src} → {dst}  ({img.width}x{img.height})")

print("\nAll assets ready. You can now build the exe.")
```

**PyInstaller spec — custom build pipeline:**
```python
# TreasuryPledge.spec — excludes bloat, keeps only required encodings
a = Analysis(
    ['main.py'],
    pathex=[],
    binaries=[],
    datas=[('logo_header.png', '.'), ('logo_bottom.png', '.'), ('ndp_bottom.png', '.')],
    hiddenimports=[],
    hookspath=[],
    hooksconfig={},
    runtime_hooks=[],
    excludes=[
        'matplotlib', 'numpy', 'pandas', 'scipy', 'tkinter.test',
        'unittest', 'pydoc', 'email', 'http', 'xml', 'html',
        'lib2to3', 'distutils', 'pkg_resources',
    ],
    win_no_prefer_redirects=False,
    win_private_assemblies=False,
    noarchive=False,
)
```

**Distribution strategy:**
- Silent deployment across government workstations
- Triggers 3x daily (morning, midday, afternoon)
- Popup contains direct link to portal — one click to current content
- No browser bookmark required; no email dependency

---

### 🗂️ Headless CMS Architecture

JSON-driven content model enabling dynamic updates without code deployment or server restart.

**Content managed via admin dashboard:**
- Hero sections (title, subtitle, statistics)
- Workstream data (icon, title, description, color, progress)
- Role-specific responsibility modules
- Financial statistics and pressure indicators

```javascript
// AdminDashboard.js — Password-protected content management
function AdminDashboard() {
  const [content, setContent] = useState(null);

  useEffect(() => {
    fetch("/content/home.json")
      .then(res => res.json())
      .then(data => setContent(data));
  }, []);

  const updateHero = (field, value) => {
    setContent({
      ...content,
      hero: { ...content.hero, [field]: value }
    });
  };

  const updateStat = (index, field, value) => {
    const updatedStats = [...content.stats];
    updatedStats[index][field] = value;
    setContent({ ...content, stats: updatedStats });
  };

  // ... workstream updates, JSON preview, save logic
}
```

**Content delivery API:**
```javascript
// Express backend serves structured JSON
app.get('/content/home.json', (req, res) => {
  res.json({
    hero: { title: "...", subtitle: "..." },
    stats: [
      { number: "R12.4B", label: "Recovery Target" },
      { number: "5", label: "Workstreams" },
      // ...
    ],
    workstreams: [ /* ... */ ]
  });
});
```

**Why this matters:** Non-technical treasury staff update content in real-time. No developer required. No deployment needed.

---

### 🎨 Interactive Information Portal

Multi-page React SPA with performance-optimized animations and mobile-first responsive design.

**Animated components:**
- Scroll-triggered fade-ins and slide animations
- Expandable workstream cards with progress indicators
- Timeline visualizations for implementation milestones
- Risk matrix displays with color-coded severity
- Sticky navigation with animated page transitions
- Floating back button for deep navigation

```javascript
// Home.js — Scroll-triggered animations with staggered card reveals
function Home() {
  const [visibleCards, setVisibleCards] = useState([]);
  const [content, setContent] = useState(null);

  useEffect(() => {
    fetch("/content/home.json")
      .then(res => res.json())
      .then(data => setContent(data));
  }, []);

  useEffect(() => {
    const timer = setTimeout(() => setVisibleCards([0,1,2,3,4,5]), 100);
    return () => clearTimeout(timer);
  }, []);

  return (
    <div className="container">
      <section className="content-section fade-in">
        <h2 className="bounce-in">{content.hero.title}</h2>
        <p className="slide-in-right">{content.hero.subtitle}</p>

        <div className="stats-grid">
          {content.stats.map((stat, index) => (
            <div key={index} className="stat-card" 
                 style={{ animationDelay: `${index * 0.2}s` }}>
              <span className="stat-number">{stat.number}</span>
              <span className="stat-label">{stat.label}</span>
            </div>
          ))}
        </div>

        <div className="card-grid">
          {content.workstreams.map((stream, index) => (
            <div key={index} 
                 className={`card ${visibleCards.includes(index) ? 'visible' : ''}`}
                 style={{ borderTopColor: stream.color, animationDelay: `${index * 0.1}s` }}>
              <div className="icon" style={{ fontSize: '2.5em' }}>{stream.icon}</div>
              <h4 style={{ color: stream.color }}>{stream.title}</h4>
              <p>{stream.description}</p>
              <div className="progress-bar">
                <div className="progress-fill" 
                     style={{ background: `linear-gradient(90deg, ${stream.color}, ${stream.color}99)`,
                              animationDelay: `${index * 0.2}s` }} />
              </div>
            </div>
          ))}
        </div>
      </section>
    </div>
  );
}
```

---

### 📊 Financial Data Visualization

Interactive bar charts, pressure indicators, and unauthorized expenditure tracking with animated rendering and expandable root cause analysis sections.

**About page features:**
- Animated bar charts showing budget allocations vs. actuals
- Pressure indicators (traffic-light system) for fiscal health metrics
- Expandable root cause analysis cards for unauthorized expenditure
- Scroll-triggered reveal animations for data storytelling

---

### 🏛️ Role-Based Information Architecture

"Our Role" module mapping department-specific responsibilities:

- Critical success factors per department
- Implementation timelines with milestone tracking
- Risk mitigation strategies
- Animated shape reveals for engagement

**Workstreams page:**
- Expandable category cards (Fiscal, Governance, Infrastructure, etc.)
- Department grid layouts
- Initiative cards with status indicators
- Risk matrices with color-coded severity levels

---

## Technology Stack

| Layer | Technology |
|---|---|
| Frontend Portal | React.js, React Router, CSS3 Animations |
| CMS Backend | Express.js, JSON file-based content store |
| Desktop App | Python, Tkinter, PIL |
| Build Pipeline | PyInstaller, custom `.spec` configuration |
| Asset Optimization | Pillow (PIL) image preprocessing |
| Deployment | Windows Server 2019, IIS, silent workstation distribution |

---

## Deployment Architecture

```
KZN Treasury Network
├── Windows Server 2019 (IIS)
│   └── React Portal + Express CMS API
│       └── /content/*.json  (headless CMS data)
│
└── Government Workstations (Windows 10/11)
    └── TreasuryPledge.exe (PyInstaller build)
        ├── Triggers 3x daily via Windows Task Scheduler
        ├── Popup with treasury branding + portal link
        └── One-click navigation to current content
```

---

## Key Metrics

- **100% workforce coverage** — Desktop app ensures every employee receives updates regardless of email habits
- **3x daily engagement** — Morning, midday, afternoon popup cycles
- **Zero developer dependency** for content updates — Treasury staff manage CMS directly
- **5 workstreams** tracked with real-time progress indicators
- **Province-wide deployment** across all KZN Treasury departments

---

## What I Learned

- **Cross-platform desktop development:** Building native-feeling apps with Tkinter and optimizing PyInstaller builds
- **Build pipeline engineering:** Stripping dependencies, preprocessing assets, minimizing executable footprint
- **Headless CMS design:** JSON-driven content architecture enabling non-technical content management
- **Animation performance:** CSS3 transforms, intersection observers, and staggered reveals without jank
- **Enterprise distribution:** Silent deployment strategies for government workstation fleets
- **Stakeholder handover:** Documenting systems so non-developers can own content management

---

## Recognition

**Certificate of Recognition — KZN Provincial Treasury**

Awarded by the **Head of Department (Carol Coetzee)** for innovative system design and delivery of the Online Assessment Management Platform and KZN Provincial Financial Recovery Plan Communication System. Recognized for end-to-end architecture, production deployment, and stakeholder impact across multiple government departments.

---

*Built during Software Engineer internship at KZN Provincial Treasury | 2025*

