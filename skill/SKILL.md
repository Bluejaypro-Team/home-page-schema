---
name: home-page-schema
description: >-
  Advanced JSON-LD Schema Markup builder for local business HOME PAGES.
  Use this skill when the user asks to create, generate, or build a schema markup
  for a local business website homepage. Produces a complete 6-block JSON-LD schema
  (WebPage, Service, LocalBusiness, Article, Place, FAQPage) mirroring the 75degreeac.com
  advanced schema pattern. Includes driving direction Maps (named + zip code),
  GeoCircle areaServed, postal code Map entries, TouristAttraction mentions,
  knowsAbout entities, Google Maps CID integration, and FAQPage with real Q&A
  scraped live from the business website.
---

# Home Page Schema Builder

This skill builds a **6-block advanced JSON-LD schema** for any local business homepage.
It follows the exact structure used by `75degreeac.com/houston-tx/` as the master template.

Blocks: WebPage → Service → LocalBusiness → Article → Place → FAQPage

---

## Required Information to Collect First

Before building, gather the following from the user:

| Field | Example |
|---|---|
| Business Name | Happy Homes Permanent Lighting |
| Street Address | 5154 Auburn Blvd |
| City | Sacramento |
| State | California |
| Zip Code | 95841 |
| Phone | +19166808114 |
| Email | info@example.com |
| Website URL | https://happyhomeslighting.com/ |
| GMB kgmid | /g/11mzm2ks8z |
| GMB CID | 7517994891832405607 |
| Business Category | ElectricalContractor / LightingContractor |
| Services (list all page URLs) | /permanent-lighting/, /commercial-lighting/ etc. |
| Logo URL | From WordPress media or site header |
| Review Count and Rating | 5.0 / 300+ |
| Opening Hours | Mon-Sun 08:00-18:00 |

### CID বের করার নিয়ম:
Google-এ business search করুন. Knowledge Panel-এ "Share" click করুন. URL-এ cid=XXXXXXXXX দেখবেন.

---

## Scraping Checklist (Data Gathering)

Before writing schema, scrape these URLs:

1. https://[domain]/ — Homepage (services, tagline, hours)
2. https://[domain]/about-us/ — About page (mission, experience years, email)
3. https://[domain]/contact-us/ — Contact (NAP, email)
4. https://[domain]/sitemap.xml — All service page URLs
5. Each service page — for description content AND for FAQ accordion sections

NOTE: If a page returns 404, exclude it from relatedLink and offers.

FAQ Scraping — grep for accordion headings:
Select-String -Path [scraped_file] -Pattern 'exad-accordion-heading|How |What |Why |Can |Do |Is |Are ' | Select-Object LineNumber, Line

---

## 6-Block Schema Structure

### Block 1 — WebPage

Key fields:
- @id: "https://[domain]/#location[city]"  (all lowercase, no page slug)
- headline: "[City], [State] - [Primary Keyword]"
- publisher.address: MUST be PostalAddress object (not plain string)
- hasPart: points to #article anchor
- mainEntityOfPage: points to #location[city] anchor

### Block 2 — Service

- Minimum 15 Offer items
- Core services + extended keyword variants
- Example extras: Outdoor Lighting, Landscape Lighting, Security Lighting, Smart Lighting, LED Retrofit, Accent Lighting, Motion Sensor Lighting

### Block 3 — LocalBusiness (Main Entity Block)

Key subfields:
- areaServed: GeoCircle with ALL area zip codes in postalCode array
- containsPlace.subjectOf: 
  a) Named location Maps (neighborhoods/cities, 20+)
  b) Postal code Maps — one per zip code in service area
- additionalType[0]: "https://www.google.com/maps?cid=[CID]"
- mainEntityOfPage: "https://www.google.com/maps?cid=[CID]"

### Map Entry Formats

Named Location Map:
{
  "@type": "Map",
  "name": "How to reach us from [Location Name]",
  "url": "https://www.google.com/maps/dir/[Location+Name]/[Street+Address]",
  "description": "From [Location Name], [State], USA\n  to [Business Name]"
}

Postal Code Map:
{
  "@type": "Map",
  "name": "How to reach us from [ZIP], [City], CA",
  "url": "https://www.google.com/maps/dir/[ZIP]+[City]+CA/[Street+Address]",
  "description": "From [ZIP], [City], California, USA\n  to [Business Name]"
}

PowerShell bulk generation for zip codes:
$zips = @("95811","95814", ...)
$n = "`n"
$zipBlock = ""
foreach ($zip in $zips) {
    $zipBlock += ",$n               {$n `"@type`": `"Map`",$n `"name`": `"How to reach us from $zip, Sacramento, CA`",$n `"url`": `"https://www.google.com/maps/dir/$zip+Sacramento+CA/5154+Auburn+Blvd,+Sacramento,+CA+95841`",$n `"description`":`"From $zip, Sacramento, California, USA$n to [Business Name]`"$n }"
}

Then use $content.Replace($target, $replacement) to insert after the last named Map entry.

### Block 4 — Article

- @id: "[domain]/#article"
- articleBody: Cover ALL services (both core + extended). City overview paragraph FIRST.
- image: Use a service/staff photo URL — NOT the logo
- mentions: 10 TouristAttraction items with Google Maps short URLs (maps.app.goo.gl)

### Block 5 — Place (TransitMap anchor)

{
  "@context": "https://schema.org/",
  "@type": "Place",
  "@id": "[domain]/#drivingdirection",
  "name": "Driving Direction",
  "hasMap": {
    "@type": "Map",
    "mapType": { "@type": "TransitMap", "@id": "https://schema.org/TransitMap" }
  }
}

### Block 6 — FAQPage

Scrape real FAQ questions from the business website (accordion sections, FAQ sections).
Minimum 8 questions. Add a bonus 10th generic question about pricing/cost.

Structure:
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "@id": "[domain]/#faqpage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "[Exact question from website]",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "[Full answer — clean text, no HTML tags]"
      }
    }
    /* repeat for each Q&A */
  ]
}

FAQ Rules:
- Scrape questions from real accordion/FAQ sections on service pages
- Strip all HTML tags from answer text before inserting
- Last Q&A should always be a cost/pricing question for the city
- @id anchor: #faqpage (homepage URL, no slug)
- Append as the LAST block in the schema file

Google SERP benefit: FAQ rich snippets appear directly in search results

---

## Anchor ID Naming Rules

| Anchor | Format | Example |
|---|---|---|
| WebPage ID | #location[city] (lowercase) | #locationsacramento |
| Business ID | #[BusinessShortName] | #LightingBusiness |
| Article ID | #article | #article |
| Driving ID | #drivingdirection | #drivingdirection |
| Logo ID | #logo | #logo |
| Website ID | #website | #website |
| FAQ ID | #faqpage | #faqpage |

CRITICAL RULE — Homepage schema:
- CORRECT: https://domain.com/#locationsacramento
- WRONG: https://domain.com/sacramento-ca/#locationsacramento
All @id anchors use homepage URL — NO page slug in the URL path.

---

## publisher.address Rule (Critical)

ALWAYS use PostalAddress object:
"address": {
  "@type": "PostalAddress",
  "streetAddress": "5154 Auburn Blvd",
  "addressLocality": "Sacramento",
  "addressRegion": "California",
  "postalCode": "95841",
  "addressCountry": "US"
}

NEVER use plain string — Google Rich Results Test will show 4 warnings.

---

## Quality Checklist Before Delivery

### Core Schema Blocks
- publisher.address is a PostalAddress object (not plain string)
- mainEntityOfPage uses real CID: https://www.google.com/maps?cid=[CID]
- additionalType[0] is the CID Maps URL
- relatedLink excludes any 404 pages
- offers has minimum 15 items
- subjectOf has named Maps (20+) + zip code Maps (one per zip)
- No duplicate Map entries (verify: Select-String -Pattern 'How to reach us from' | Count)
- articleBody covers ALL services in offers list
- mentions in Article has 10 TouristAttraction items
- All @id anchors use homepage URL (no page slug)
- locationCity anchor is all lowercase (e.g., #locationsacramento)

### FAQPage Block
- FAQPage block exists as the last script block
- Minimum 8 real Q&A scraped from website
- Last question is about cost/pricing in the city
- All HTML tags stripped from answer text
- @id is [domain]/#faqpage

### Final Validation
- Google Rich Results Test: 0 errors, warnings only "optional"
- FAQPage test: https://search.google.com/test/rich-results (select FAQPage type)

---

## Validation

Test at: https://search.google.com/test/rich-results
- Green checkmark = no critical errors
- Yellow = optional fields missing (acceptable)
- Red = must fix before publishing

## Template Reference

Master template: 75degreeac_houston_schema.html
Completed example (with FAQPage): happyhomes_sacramento_schema.html
Local path: C:\Users\BlueJayPro\.gemini\antigravity-ide\brain\7b890ccf-f352-48f6-b14d-ebab1fa043af\

GitHub Repository: https://github.com/Bluejaypro-Team/home-page-schema
- templates/ — master template (75degreeac)
- completed/ — Happy Homes full schema with FAQPage
- skill/ — this SKILL.md
