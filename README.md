# SEO and Keyword Analytics in Nigerian Consumer Banking  

![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Tool](https://img.shields.io/badge/Tool-Power_BI-blue)
![Analysis](https://img.shields.io/badge/Type-SEO_&_Market_Visibility_Analysis-orange)

---

## 🎯 Objective  

This project analyzes how Nigerian consumer banks perform in Google search visibility.  
It measures which banks dominate high-value keywords like *loans*, *online banking*, and *account opening*, and links that visibility to profitability.  

Goals:  
1. Identify which banks appear most across 80+ banking keywords.  
2. Measure keyword search volumes and visibility share per bank.  
3. Compare SEO visibility against reported profit (PAT).  
4. Visualize all insights in a public Power BI dashboard.  

The outcome: a clear view of which banks lead in SEO reach and how digital presence aligns with real business strength.

---
## Data Collection

### Keyword Selection
- Built an **85-keyword seed list** focused on consumer banking.  
- Each keyword localized with “bank” and “Nigeria” for relevance.  

**Examples:**  
`loan` → `loan Nigeria`  
`mobile banking` → `mobile banking Nigeria`  

---

### Data Source
- Tool used: **SerpApi** via **Google Apps Script**.  
- Extracted **top 20 organic Google results** per keyword.  

**Captured fields:**  
- `Banks_Appearing` — bank names appearing in titles/snippets.  
- `Rank_Position` — order in which banks appear on SERP.  

**Automation:**  
- Google Apps Script scheduled and controlled queries.  
- Ensured queries stayed within SerpApi’s **250-query monthly limit**.

---

### Enrichment Metrics
- **Average Monthly Searches** (Google Keyword Planner / Ubersuggest).  
- **Competition Level** and **Top of Page Bid (Low/High)** for paid search insights.  
- **PAT / Profit (₦B)** from bank annual reports or public estimates.  

**Data Cleaning & Transformation:**  
- Normalized and standardized bank names.  
- Removed duplicates and trimmed whitespace.  
- Created `Keyword_short` for simplified labeling.  
- Developed calculated measures and KPIs in **Power BI** for:  
  - Keyword competitiveness  
  - Ranking patterns  
  - Bank visibility distribution  

---

## Core Script (seo_data_scraper.js)
```javascript
// Top 20 Nigerian banks
const banks = [
  "Access Bank", "Zenith Bank", "GTBank", "UBA", "First Bank", 
  "Fidelity Bank", "FCMB", "Stanbic IBTC", "Union Bank", "Polaris Bank",
  "Sterling Bank", "Wema Bank", "Keystone Bank", "Providus Bank", "SunTrust Bank",
  "Jaiz Bank", "Unity Bank", "Globus Bank", "Lotus Bank", "Nova Merchant Bank"
];

const BATCH_SIZE = 10;  // number of keywords per run
const SLEEP_MS = 2000;  // pause 2s between requests

// Fetch banks appearing and their rank positions for one keyword
function getBanksAndRank(keyword) {
  const key = "YOUR_SERPAPI_KEY"; // replace with your key
  const url = `https://serpapi.com/search.json?q=${encodeURIComponent(keyword)}&engine=google&num=20&api_key=${key}`;
  const response = UrlFetchApp.fetch(url);
  const json = JSON.parse(response.getContentText());

  const results = json.organic_results || [];
  let appearingBanks = [];
  let ranks = [];

  results.forEach((res, index) => {
    const title = res.title.toLowerCase();
    banks.forEach(bank => {
      if (title.includes(bank.toLowerCase()) && !appearingBanks.includes(bank)) {
        appearingBanks.push(bank);
        ranks.push(index + 1); // rank starts at 1
      }
    });
  });

  Logger.log(`Keyword: ${keyword} => Banks: ${appearingBanks.join(", ")} | Ranks: ${ranks.join(", ")}`);
  return [appearingBanks.join(", "), ranks.join(", ")];
}

// Run batch for keywords in the sheet
function runSEODataBatch() {
  const startRow = 1; // starting keyword index
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const keywords = sheet.getRange("A2:A").getValues().flat().filter(Boolean);

  // Set headers if empty
  if (!sheet.getRange("B1").getValue()) sheet.getRange(1, 2).setValue("Banks_Appearing");
  if (!sheet.getRange("C1").getValue()) sheet.getRange(1, 3).setValue("Rank_Position");

  const endRow = Math.min(startRow + BATCH_SIZE - 1, keywords.length);

  for (let i = startRow - 1; i < endRow; i++) {
    const [banksAppearing, ranks] = getBanksAndRank(keywords[i]);
    sheet.getRange(i + 2, 2).setValue(banksAppearing);
    sheet.getRange(i + 2, 3).setValue(ranks);
    Utilities.sleep(SLEEP_MS); // avoid rate limit
  }

  Logger.log(`Processed keywords ${startRow} to ${endRow}.`);
}
```
---

## Dataset

**File:** `Keyword_SEO_Banking_Nigeria.csv`  

**Columns include:**  
- `Keyword`  
- `Keyword_short`  
- `Banks_Appearing`  
- `Rank_Position`  
- `Avg. monthly searches`  
- `Competition`  
- `Top of Page Bid (Low/High)`  
- `PAT / Profit (₦B)`  

---

## Power BI Dashboard

**Public Link:** [View on Power BI Public](#)  

**Key Visuals:**  
- Keyword Count per Bank  
- Keyword Visibility Share (%) by Bank  
- Top 10 Keywords by Monthly Search  
- Keyword Visibility vs Profit (scatter)  
- Rank Heatmap  
- Top of Page Bid (High Range) per Bank  

**Interactive Features:**  
- Slicers for Banks, Avg Monthly Searches, and Rank Position  

---

## Analysis & Insights

- **UBA** dominates in both visibility share (32%) and raw keyword count (30).  
- **Access Bank** and **GTBank** follow with approximately 11% each.  
- Scatter plot indicates a **weak positive correlation** between SEO visibility and profit (PAT).  
- High-profit banks with low visibility represent opportunities for **targeted SEO content expansion**.  
- High-visibility banks with low profit may need **UX or conversion optimization** to better capitalize on traffic.

---
## Recommendations

- High-PAT, low-SEO banks: invest in **keyword-targeted content**.  
- High-SEO, low-PAT banks: optimize **conversion funnels**.  
- Build **content clusters** for under-served high-volume terms.  
- Track **SERP ranks monthly** to validate SEO ROI.  

---

## Limitations

- SerpApi capped at **250 queries/month**.  
- **SERP positions fluctuate** daily.  
- Financial data partly **estimated**.  
- CPC and competition are **relative**, not absolute.  

---

## Conclusion

This analysis explores the link between **SEO visibility** and **profit performance** across Nigerian consumer banks.  
**UBA** leads with 32% visibility share and the highest keyword presence.  
A weak positive correlation exists between SEO visibility and profitability, suggesting that while search visibility matters, it does not fully predict financial outcomes.  
Banks can leverage these findings to refine **content strategy**, boost **brand reach**, and improve **conversion efficiency**.  

**Full Report:** [Nigerian_Banks_SEO_Profit_Report.pdf](Nigerian_Banks_SEO_Profit_Report.pdf)  
**Dashboard:** [View Power BI Report](https://app.powerbi.com/view?r=eyJrIjoiZmVhOGIzOTItYjRhZi00NDMwLTk2YWItMWQ0NTdiNzg4YjVmIiwidCI6IjU3ODlhOGIxLWFjN2MtNDMxZS05YTQyLWJlOTk0NTNjNWIzMCJ9)  

---

## Author & Contributors

**Author:** [Lawal Mayowa](https://www.linkedin.com/in/lawal-mayowa-160bb930b/)  
**Dataset Preparation & Script:** `seo_data_scraper.js`  
**Power BI Dashboard:** `SEO_Dashboard.pbix`  
**Data Source:** SerpApi, Google Keyword Planner, Ubersuggest  

---


