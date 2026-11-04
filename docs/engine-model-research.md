# Engine model — research findings

Research for [issue #18](https://github.com/elmofromok/MoMadness/issues/18).
The decisions are in
[ADR-0005](adr/0005-the-engine-model.md). This file holds the starting numbers
and where they came from.

Each figure is marked as a manufacturer claim, a measured dyno, or an estimate.
Two sources were seen only as search summaries because the pages did not load,
and are marked as such. Check those before relying on them.

## Torque and power

Dirt Rider rear-wheel dyno measurements.

| Bike / year | Peak power | Peak torque | Source |
|---|---|---|---|
| CRF450R 2026 | 52.6 hp | 33.7 lb-ft (45.7 Nm) | [Dirt Rider](https://www.dirtrider.com/tests/honda-crf450r-dyno-test-2026), no peak RPM given |
| CRF450R 2023 | 52.5 hp @ 9,100 | 35.1 lb-ft (47.6 Nm) @ 6,600 | [Dirt Rider](https://dirtrider.com/tests/honda-crf450r-dyno-test-2023) |
| CRF450R 2022 | 53.8 hp @ 9,300 | 34.1 lb-ft @ 7,200 | [Dirt Rider](https://dirtrider.com/story/tests/2022-honda-crf450r-dyno-test), summary only |
| KTM 450 SX-F 2022 | 55.0 hp @ 9,400 | 34.3 lb-ft @ 7,500 | [Dirt Rider](https://dirtrider.com/story/tests/2022-ktm-450-sx-f-dyno-test) |
| KTM 450 SX-F 2025 | 53.2 hp @ 9,300 | 32.9 lb-ft | [Dirt Rider](https://dirtrider.com/tests/ktm-450-sx-f-dyno-test-2025) |

Shape: torque peaks around 6,600 to 7,500 RPM at about 46 to 48 Nm, and power
around 9,100 to 9,500 RPM. adbmag's 2023 KTM test (summary only) describes
torque reaching 48 Nm just before 7,000 RPM and falling off slowly to 11,000.
[adbmag](https://adbmag.com.au/editorial/dyno-test-2023-ktm-450-sx-f-vs-300-sx/)

No RPM-by-RPM table was found; values exist only in chart images. **Estimate**
for a usable starting shape: a plateau of about 85 to 100% of peak torque from
5,000 to 9,000 RPM, falling to about 70% at the limiter.

## Idle and rev limit

- Idle: 2,000 ± 100 RPM, 2022 CRF450R owner's manual (manufacturer).
  [manualslib p.78](https://manualslib.com/manual/2423458/Honda-Crf450r-2022.html?page=78)
- Rev limit: about 11,500 RPM on the 2025 CRF450RWE
  ([Dirt Rider](https://www.dirtrider.com/tests/honda-crf450rwe-dyno-test-2025),
  summary only) and about 11,000 RPM on the KTM (adbmag). Observed on dynos;
  no manufacturer figure found.

## Gearbox (CRF450R 2022–2025)

From Honda's manual and
[2024 spec sheet](https://hondanews.com/en-US/powersports/releases/2024-honda-crf450r-specifications)
(manufacturer). The manual's layout is scrambled, so the gear order is inferred.

- 5 speeds. Primary reduction 2.357.
- 1st 2.133, 2nd 1.706, 3rd 1.421, 4th 1.211, 5th 1.043.
- Final drive 3.769 (13/49 sprockets, #520 chain).
- Tires 80/100-21 front, 120/80-19 rear.

**Estimate:** rear rolling diameter about 675 mm, circumference about 2.12 m.
Knobs and squat shift this by a few percent.

Road speed at 11,500 RPM (calculated):

| Gear | Overall ratio | Speed |
|---|---|---|
| 1st | 18.95 | 77 km/h |
| 2nd | 15.16 | 97 km/h |
| 3rd | 12.62 | 116 km/h |
| 4th | 10.76 | 136 km/h |
| 5th | 9.27 | 158 km/h |

## Rotating inertia

- **Crank:** no published moment of inertia for any 450 MX crank. Trials-bike
  flywheels run about 0.0074 to 0.0114 kg·m²
  ([electricmotiontech](https://sites.google.com/view/electricmotiontech-com/home/what-makes-a-good-trials-motor/flywheel-inertia)).
  **Estimate** for a 450's crank, flywheel and rotor: 0.01 to 0.02 kg·m² at
  crank speed. Seen from the rear wheel it is multiplied by the overall ratio
  squared. [SAE 971060](https://papers.sae.org/publications/technical-papers/content/971060/)
  covers crank inertia and motorcycle behaviour; not read.
- **Direction:** conventional transverse singles spin the crank the same way as
  the wheels.
  [Visordown](https://www.visordown.com/features/guides/what-counter-rotating-crankshaft-and-why-use-one)
  No production 450 MX bike with a reversed crank was found. The YZ450F reverses
  its cylinder head, not its crank.
  [Canada Moto Guide](https://canadamotoguide.com/2009/10/08/yamahas-yz450f-reversed-head/)
  This rests on not finding one.
- **Wheels:** nothing published. **Estimate:** rear assembly 11 to 13 kg, front
  8 to 10 kg; as a ring at about 0.30 m, rear inertia about 0.8 to 1.0 kg·m².

## Not verified

Any manufacturer rev limit, RPM-by-RPM torque values, the exact gear order, and
every inertia and wheel mass figure.
