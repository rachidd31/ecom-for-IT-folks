
# Traffic — Engineer's Cheat Sheet

## 1. Glossary

| Term | Definition |
|---|---|
| **Organic Traffic** | Unpaid visits from search engines (SEO-driven). |
| **Paid Traffic** | Visits bought via ads (PPC, social ads). |
| **Direct Traffic** | User typed URL or used bookmark — no referring source. |
| **Referral Traffic** | Arrived via link on another site (blog, partner). |
| **Social Traffic** | Arrived from social platforms (organic or paid). |
| **SEO** | Search Engine Optimization — ranking higher organically. |
| **SEM** | Search Engine Marketing — paid search ads (Google Ads). |
| **PPC** | Pay-Per-Click — you pay per ad click. |
| **CPC** | Cost Per Click — price paid for one click. |
| **CPM** | Cost Per Mille — price per 1,000 ad impressions. |
| **CTR** | Click-Through Rate — clicks ÷ impressions. |
| **CVR** | Conversion Rate — conversions ÷ visits. |
| **Bounce Rate** | % of visitors who leave after one page. |
| **UTM Parameters** | URL tags tracking source/medium/campaign. |
| **Attribution** | Model crediting which touchpoint caused a conversion (first-click, last-click, multi-touch). |
| **Retargeting** | Ads shown to users who already visited but didn't convert. |
| **Funnel** | Stages a visitor passes: awareness → consideration → conversion. |
| **CAC** | Customer Acquisition Cost — total spend ÷ new customers. |
| **LTV** | Lifetime Value — total revenue expected per customer. |
| **Landing Page Traffic** | Visits hitting a dedicated conversion-focused page (vs homepage). |
| **Referrer Header** | HTTP header telling server which page linked to this request. |

---

## 2. Brief History (Timeline)

```mermaid
timeline
    title Evolution of Web Traffic & Acquisition
    1994 : Banner ads - first clickable web ad (HotWired)
    1998 : Google launches - organic search becomes a channel
    2000 : Google AdWords - paid search (SEM) born
    2005 : Google Analytics - traffic measurement goes mainstream
    2007 : Facebook Ads Platform - social + paid social traffic
    2010 : Mobile traffic surge - smartphones shift traffic mix
    2013 : Retargeting/pixel tracking scales - Meta Pixel, Google Remarketing
    2018 : GDPR - privacy law reshapes tracking
    2021 : iOS 14.5 ATT - third-party tracking breaks down
    2020s : Cookieless + server-side tracking - CAPI, first-party data era
```

**Gist:** traffic measurement moved from *none* → *pixel-based tracking everywhere* → *privacy pushback* → *first-party/server-side tracking*.

---

## 3. Traffic Sources Map

```mermaid
flowchart LR
    U[User] --> D[Direct - typed URL]
    U --> O[Organic - Google/Bing search]
    U --> P[Paid - Google Ads/Meta Ads]
    U --> S[Social - organic posts]
    U --> R[Referral - external link]
    U --> E[Email - campaigns]

    D & O & P & S & R & E --> LP[Landing Page / Homepage]
    LP --> PDP[Product Page]
    PDP --> CART[Cart]
    CART --> CHK[Checkout]
    CHK --> CONV[Conversion]
```

---

## 4. Funnel + Attribution Flow

```mermaid
sequenceDiagram
    participant Ad as Ad Click
    participant Browser as Browser
    participant Site as Site (UTM captured)
    participant Analytics as Analytics/Pixel
    participant Server as Server (CAPI)
    participant CRM as CRM/Ad Platform

    Ad->>Browser: click with ?utm_source=...
    Browser->>Site: load landing page
    Site->>Analytics: fire pageview + UTM
    Site->>Server: log session (first-party)
    Browser->>Site: add to cart / purchase
    Server->>CRM: send conversion event (server-side)
    CRM-->>Ad: attribute conversion to campaign
```

---

## 5. Channel Comparison

| Channel | Cost Model | Speed to Results | Control | Best for |
|---|---|---|---|---|
| Organic/SEO | Free (time cost) | Slow (months) | Low (algorithm-dependent) | Long-term compounding traffic |
| Paid Search | CPC | Fast | High | High-intent buyers |
| Paid Social | CPM/CPC | Fast | High | Discovery, retargeting |
| Referral | Free/rev-share | Medium | Medium | Trust-driven conversion |
| Email | Low fixed cost | Fast | High | Retention, repeat purchase |
| Direct | Free | N/A | Low | Brand strength indicator |

---

## 6. Outline Gist (one-screen recap)

1. **Definition** → traffic = visitors arriving through a specific channel/source.
2. **History** → no tracking → pixel tracking everywhere → privacy laws → first-party/server-side tracking.
3. **Flow** → Source → Landing Page → Product Page → Cart → Checkout → Conversion, tagged via UTM, measured via Analytics + Server-side (CAPI).
4. **Modern shift** → cookieless world pushes tracking server-side and first-party.
5. **Engineer's concern** → UTM consistency, pixel + CAPI dedup, page speed (affects bounce/CVR), attribution data pipeline.
