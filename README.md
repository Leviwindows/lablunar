# LabLunar

LabLunar is an educational simulator for children aged 7 to 12. Players design and run a lunar outpost, balancing power, oxygen and radiation. Every choice has a cost: saving power at night can leave the base short of air or heat, and solar storms require sheltering. At the end, a report explains the chain of causes behind success or failure.

## How to run

1. Download or clone this repository.
2. Open `index.html` in any modern browser. No installation, account or server is needed.

## How to play

1. Design the outpost within the mass budget: solar panels, batteries, greenhouses, recycling and radiation shielding.
2. Start the mission and make daily decisions: energy saving, sheltering during storms.
3. Read the final report to see why the mission succeeded or failed.

## NASA data

| Source | Used for | Status |
|---|---|---|
| NASA DONKI (Space Weather Database) | Solar storm events | Partial: snapshot in `data/donki_snapshot.json`; live integration pending |
| NASA OSDR (Open Science Data Repository) | Radiation and health reference values | Pending: values marked "illustrative" until sourced |
| NASA Artemis mission data | Lunar environment (day and night length, radiation) | Pending: see `data/sources.md` |

All sources, access dates and the status of each value are listed in `data/sources.md`. Values marked "illustrative" are for teaching only and are not NASA measurements.

## Balance tests

Balance was checked with automated simulations of the game engine. Results are pending and will be added here.

## AI use

This project was developed with AI assistance (Claude, Anthropic) for design and code generation. The team reviewed the mechanics, the data sources and the game text.

## License

To be defined by the team.
