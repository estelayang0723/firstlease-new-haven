# FirstLease New Haven

A Yale SOM-centered housing search and lease-to-move-in assistant for international students renting in New Haven. It helps compare source-linked apartment-community plan/pricing snapshots by floorplan, listed rent, utilities, availability, and approximate commute; a local retrieval chatbot answers questions from those records. The original lease analysis and move-in checklist remain available as a second workflow.

## Run locally

1. Install Python 3.11 or newer.
2. From this folder, create and activate a virtual environment:
   - macOS/Linux: `python3 -m venv .venv` then `source .venv/bin/activate`
   - Windows PowerShell: `py -m venv .venv` then `.venv\Scripts\Activate.ps1`
3. Install dependencies: `python -m pip install -r requirements.txt`.
4. Copy `.env.example` to `.env` if using a model or live listings. Keep secret values out of Git. The app also checks the parent folder for `.env`, as required by the course runner.
5. Optional: add the course `PORTKEY_API_KEY` for lease-review AI (the default model is `gpt-6-luna`). No key is needed for housing search/chat or offline lease demo.
6. Optional local-only feature: add `RENTCAST_API_KEY` if you want live rentals. The button is manual and results are cached for 20 minutes. A SQLite counter locally reserves at most 40 attempts and failed attempts count. It is not account-wide and is not durable on hosts with ephemeral filesystems; keep this key unset on Render Free.
7. Start: `python -m uvicorn app:app --host 127.0.0.1 --port 8000`.
8. Open http://127.0.0.1:8000 and keep the terminal running while using the app.

For a fresh-clone check, follow these steps from a new clone, put the teaching staff `.env` one folder above the repo, and confirm the app opens and the offline housing chat works without a key. The optional live listing button is never called automatically.

The app also has a clearly labeled offline demonstration mode when no API key is set. Uploaded documents are processed in memory and are not saved by the server. In live mode, candidate lease excerpts (which may include dates, rent, and obligations) are sent to the configured model provider through Portkey. Redact names, signatures, account details, and addresses before uploading; do not upload unredacted personal leases.

## Deploy online with Render

This repository includes a `render.yaml` Blueprint. After you publish the repository to GitHub:

1. In Render, choose **New → Blueprint**, connect GitHub, and select this repository. Render reads `render.yaml` from the repository root.
2. Confirm the service name `firstlease-new-haven` and deploy. The config installs `requirements.txt` and starts FastAPI with Render's assigned `$PORT`.
3. The service works in offline mode without any API key. If you intentionally want live lease review, add `PORTKEY_API_KEY` in the Render service's Environment settings. Never put the key in GitHub, `render.yaml`, or screenshots. The model defaults to `gpt-6-luna`.
4. Do not add `RENTCAST_API_KEY` to a free Render service. RentCast is optional, and the app's 40-request counter is a local SQLite file. Render Free can restart or spin down and loses local files, so that counter would not be a durable billing safeguard. Without the RentCast key, the app uses its included snapshot instead.
5. After deployment, copy the service's `onrender.com` URL into this README, `report.html`, and `slides.html`. Test it in a private/incognito window and note the first request may take about a minute after idle.

Render Free web services spin down after 15 minutes without traffic and have an ephemeral filesystem. Uploaded leases are not saved by this app, but local SQLite state is not durable on that plan. See the [Render Free instance limits](https://render.com/docs/free), [FastAPI deploy guide](https://render.com/docs/deploy-fastapi), and [environment variable docs](https://render.com/docs/configure-environment-variables).

## What it does

- Filters and compares source-linked Yale Off-Campus Marketplace listings and apartment-community plan/pricing snapshots around Yale SOM; select up to four listings for a print/save-as-PDF report.
- Optionally loads active long-term rental records within a 2-mile radius through the RentCast API when the user clicks “Load live listings” and a key is configured.
- Provides a local retrieval chatbot for floorplan, rent, listed concessions, utilities, and source questions.
- Estimates lease-period cost, effective monthly cost, and move-in cash using an explicit rent, free-month, fee, and upfront-charge formula; the user verifies inputs and the estimate excludes unentered costs.
- Saves a housing shortlist and private notes in browser local storage only; saved items are not uploaded or synced.
- Generates a fixed, listing-specific landlord question checklist covering exact-unit availability, total monthly charges, move-in payments, utilities, concessions, and maintenance. Questions can be copied without a model/API call.
- Supports English and common Chinese studio/bedroom search phrasing. The housing chatbot is deterministic local retrieval and does not call an LLM or RentCast.
- Marks deals as “not listed” when no concession appears in the listing text; this is not proof that no landlord offer exists.
- Displays the source snapshot date and links out so users can confirm availability, rent, floorplan, and terms directly.
- Estimates SOM commute from the location for prototype planning; it is not a routed or guaranteed travel time.

- Extracts key dates, rent, deposit, utilities, maintenance terms, and move-in instructions.
- Shows exact supporting excerpts from the uploaded lease.
- Lists missing or unclear items as questions to confirm with the landlord.
- Builds a move-in checklist from the lease and the selected move-in date; completed items can be checked and are saved locally in this browser by move-in date.
- Optionally compares quoted lease facts for rent, utilities, deposit, and lease term beside one selected listing snapshot and its source. This is a side-by-side evidence view, not an automatic decision or legal conclusion.
- Shows model token usage when returned by the provider.

This is an educational information-organizing tool, not legal advice. It does not determine whether a lease is legal, verify a property or landlord, contact anyone, sign documents, or make payments. Users should consult qualified professionals for legal questions.

### Housing data scope

The default set is a one-time public-page snapshot checked on 2026-10-25. It combines Yale Off-Campus Marketplace sample listings with official floorplan/pricing pages for The Haven on College, Bulldog Apartments, The Winston, The Archive, The Audubon, Anthem Square 10, 254 College Street, Madison Towers, Crown Towers, Crown Court, and 18 High (New Haven Towers), Twenty Fair, 360 State, The Liberty Building, Residence on the Green (900 Chapel Tower), 38 Crown Street, The Eli, Florence Virtue, The Union, Residence Court, and Strouse Adler. Some records are plan examples, some show a price, and some pages show a specific availability status. They are not guaranteed current vacancies and this set is not an automatically refreshed inventory feed. Prices are copied as displayed; monthly fee definitions and lease terms differ by property, so verify the property page and quote before comparing. Optional RentCast results are fetched only after an explicit user action and cached for 20 minutes; that API may not supply floorplans, concessions, utilities, or a route to SOM. Yale's Off-Campus Living Guide says the university service does not inspect or endorse listed housing; users should view a unit and verify details before signing.

## Course submission checklist

- Publish this folder to GitHub and add that URL at the top of this README, `report.html`, and `slides.html`. Make the repo public or add `zlisto` as a collaborator.
- Record real token counts from a live run; do not report estimates as measurements. Offline mode has zero model/API tokens.
- From a clean clone, follow the run instructions with the course `.env` one folder above the repo. Capture app screenshots and a real safety-defense run; embed screenshots in both HTML deliverables.
- Ask a target user to try a defined task and report only observed timing, accuracy, and feedback; do not fill these values with guesses.
- Refresh each listing directly from its source before describing it as currently available or advertising a deal.
- Print-check `report.html` (10–15 pages) and confirm `slides.html` has at most 10 slides.
