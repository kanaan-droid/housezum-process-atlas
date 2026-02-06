# Agent License Numbers by Country

Research on how real estate agent credentials work in different markets. This will inform our agent profile validation and listing source verification features.

## Summary Table

| Country | Individual License | Agency/Business License | Verification Method |
|---------|-------------------|------------------------|---------------------|
| **UAE (Dubai)** | BRN (Broker Registration Number) | ORN (Office Registration Number) | Dubai REST App, DLD API |
| **UAE (Abu Dhabi)** | BLN (Broker License Number) | BLN (Broker License Number) | ADREC |
| **France** | Carte Professionnelle or RSAC | SIRET / SIREN | API Recherche d'Entreprises |
| **USA** | State License Number | Brokerage License | State-specific MLS/DRE |

---

## 1. United Arab Emirates (UAE)

Real estate in the UAE is regulated at the **Emirate level**.

### Dubai

- **BRN (Broker Registration Number):** Individual agent's license. Every licensed agent must have a RERA Broker ID Card displaying this number.
- **ORN (Office Registration Number):** Registration number for the real estate agency (firm) itself.
- **Permit Number (Trakheesi):** Every property advertisement must display a unique Trakheesi Permit Number confirming the listing is legally approved for marketing.

**Key Point:** In Dubai, agents cannot work as freelancers. They must be employed by a licensed agency. An agent has their own BRN but operates under their company's ORN.

### Abu Dhabi

- **BLN (Broker License Number):** Agents and firms must register with the Abu Dhabi Real Estate Centre (ADREC) to obtain a BLN.

### Verification

- **Dubai REST App (Official):** Primary tool for verifying agents. Enter BRN (Agent) or ORN (Office).
- **DLD Website:** Verify License and Permits portal for licensed brokers and offices.
- **Madmoun Service:** Scan QR Code on advertisements to verify listing permits.

### API Access

- **Dubai Brokers API:** Real-time access to broker card and office information.
  - Cost: AED 30,000 + VAT annually
  - Requires registered DLD business account
- **Dubai Pulse Open Data:** Free alternative with limited data via `dld_brokers-open-api`
  - Documentation: [Dubai Pulse API Guide](https://www.dubaipulse.gov.ae)

---

## 2. France

France does not have a centralized MLS system. Real estate agents are regulated by the **Hoguet Law**.

### License Types

- **Carte Professionnelle (Professional Card):** Mandatory license to practice real estate. Issued by CCI (Chamber of Commerce and Industry), renewed every 3 years.
- **SIRET / SIREN Number:** All real estate professionals must be registered in the French national business directory.
  - SIREN: 9-digit business identification number
  - SIRET: 14-digit number (SIREN + 5-digit location code)
- **RSAC Number:** Independent agents (agents commerciaux) who work under an agency's license must have a registration from the Special Register of Commercial Agents.

### Verification

- **L'Annuaire des Entreprises:** National Business Directory search by SIREN/SIRET
- **INPI Data Portals:** National Registry of Companies (RNE) for commercial agents (RSAC)
- **CCI Professional Card Search:** Verify Carte Professionnelle

### API Access (FREE)

- **API Recherche d'Entreprises:** Primary API for searching National Business Directory
  - Endpoints:
    - `GET /search` - Search by name or address
    - `GET /entreprise/{siren}` - Full details by SIREN
    - `GET /etablissement/{siret}` - Branch details
  - Documentation: [API Recherche d'Entreprises](https://recherche-entreprises.api.gouv.fr)

### Example: Python Verification

```python
import requests

def verify_french_business(siren_number):
    url = f"https://recherche-entreprises.api.gouv.fr/search?q={siren_number}"
    
    response = requests.get(url)
    response.raise_for_status()
    
    data = response.json()
    
    if data['results']:
        business = data['results'][0]
        name = business.get('nom_complet')
        siren = business.get('siren')
        status = business.get('etat_administratif')  # 'A' for Active
        
        print(f"✅ Found: {name}")
        print(f"   SIREN: {siren}")
        print(f"   Status: {'Active' if status == 'A' else 'Inactive'}")
    else:
        print("❌ No business found with that SIREN.")

# Example
verify_french_business("803849453")
```

---

## 3. USA

- **State License Number:** Each state has its own real estate commission (e.g., California DRE)
- **MLS Number:** Not a license — it's a listing identifier assigned by the local MLS
- Verification varies by state

---

## Implementation Notes for HouseZum

### Agent Profile Fields (Future)

Based on this research, we may need country-specific license fields:

| Country | Field Name | Example |
|---------|-----------|---------|
| UAE (Dubai) | `brn_number` | "12345" |
| UAE (Dubai) | `orn_number` | "67890" |
| UAE (Abu Dhabi) | `bln_number` | "AD-12345" |
| France | `siret_number` | "80384945300015" |
| France | `rsac_number` | "123456789" |
| France | `carte_pro_number` | "CPI 7501 2019 000 012345" |
| USA | `license_number` | "DRE# 01234567" |

### Verification Priority

1. **Phase 1:** Manual entry of license numbers in agent profile
2. **Phase 2:** Display license on shared listings for credibility
3. **Phase 3:** API verification (France free, UAE paid)

### API Cost Comparison

| Country | API | Cost | Access |
|---------|-----|------|--------|
| France | API Recherche d'Entreprises | **Free** | Public |
| UAE (Dubai) | Dubai Brokers API | AED 30,000/year (~$8,200) | Business account required |
| UAE (Dubai) | Dubai Pulse Open Data | **Free** | Limited data |

---

## References

- [Dubai Land Department](https://www.dubailand.gov.ae)
- [Dubai REST App](https://www.dubailand.gov.ae/en/eServices/Dubai-REST-App/)
- [API Recherche d'Entreprises](https://recherche-entreprises.api.gouv.fr)
- [INPI RNE](https://data.inpi.fr)
- [Data.gouv.fr](https://www.data.gouv.fr)

---

*Last updated: 2026-02-06*
