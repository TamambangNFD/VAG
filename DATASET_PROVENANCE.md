# Vision Aid Glasses — Dataset Provenance

Tracks the source, licensing, and collection method of any dataset used for
training or fine-tuning a model in this project — required by the roadmap
for face data (Phase 7) and currency data (Phase 10), and applicable to any
future phase that trains or fine-tunes on collected data.

## Currency — XAF Reference Images (Phase 10)

| Field | Status |
|---|---|
| Purpose | Few-shot reference embeddings for Central African CFA franc (XAF) denomination classification, extending Microsoft's BankNote-Net encoder (see `MODEL_REGISTRY.md`) to a currency outside its base 17. |
| Source | **Not yet collected.** Must be real photographs of genuine XAF banknotes/coins, taken specifically for this purpose, or imagery whose license explicitly permits ML feature-extraction use. |
| Explicitly prohibited sources | Stock photography sites (Shutterstock, iStock, Dreamstime, Envato, and similar) — their standard licenses do not grant rights for this use, regardless of how easy they are to find via search. |
| Collection method | **TBD — to be filled in once collected.** Should record: who photographed the notes/coins, approximately when, how many images per denomination, and under what lighting/condition variety. |
| Consent/rights | Photographing one's own or legitimately obtained currency for a legitimate assistive-technology purpose does not raise the same individual-consent issue as photographing a person's face (Phase 7) — there is no data subject whose consent is needed the way an enrolled person's is. The relevant right to confirm is copyright/license clearance if any third-party image source is ever used instead of original photography. |
| Storage | Reference embeddings (not raw images) are stored in `vision/src/main/assets/currency_reference_embeddings.json`, per `MODEL_REGISTRY.md`. |

**This entry must be completed with real details before or as part of
actually collecting the reference images** — do not leave it as a
placeholder once data collection happens; an undocumented dataset is
exactly what this file exists to prevent.

## Face — Enrollment Data (Phase 7)

Not a training dataset in the traditional sense — see `PRIVACY.md` for the
full handling of user-enrolled face data, which is collected per-user at
runtime with structural consent enforcement, not a bulk dataset collected
by the development team. No pretrained face-*identification* model is
integrated at all (see `MODEL_REGISTRY.md`'s "Face Recognition / Identity
Embedding" section) — so there is no training dataset to document here
beyond the enrollment flow itself.
