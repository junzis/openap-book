# API Reference for OpenAP Handbook - Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add a "Part III: API Reference" to the OpenAP Handbook (openap.dev) with one `.qmd` page per module, covering every public class, method, and function with signatures, parameter tables (including units), return types, and minimal runnable examples.

**Architecture:** New `api/` directory in the Quarto book with 13 `.qmd` files. Each page follows a consistent format: intro sentence, constructor (if class), methods with parameter tables, and a short runnable example per method. Update `_quarto.yml` to add the new part to the book sidebar.

**Tech Stack:** Quarto (`.qmd` files), Python code blocks (`{python}`), existing openap library

---

### Task 1: Update `_quarto.yml` to add Part III

**Files:**
- Modify: `_quarto.yml`

**Step 1: Add the API reference part after the optimize section**

Insert a new `part` block between the optimize chapters and `references.qmd`:

```yaml
    - part: api/index.qmd
      chapters:
        - api/prop.qmd
        - api/thrust.qmd
        - api/drag.qmd
        - api/fuelflow.qmd
        - api/emission.qmd
        - api/wrap.qmd
        - api/flightphase.qmd
        - api/flightgenerator.qmd
        - api/aero.qmd
        - api/geo.qmd
        - api/contrail.qmd
        - api/backends.qmd
```

**Step 2: Commit**

```bash
git add _quarto.yml
git commit -m "docs: add Part III API Reference to book config"
```

---

### Task 2: Create `api/index.qmd` (Part III intro)

**Files:**
- Create: `api/index.qmd`

**Content:** Brief intro explaining what the API reference covers, how units work (kt, ft, fpm for flight params; SI for physics), and that all classes accept an optional `backend` parameter. Keep it short — 10-15 lines.

**Step 1: Write the file**

```markdown
# 📖 API Reference {.unnumbered}

This section provides a complete reference for every public module, class, and function in OpenAP.

## Conventions

- **Speed**: knots (kt) for airspeed inputs in performance models
- **Altitude**: feet (ft) for altitude inputs in performance models
- **Vertical rate**: feet per minute (ft/min, fpm)
- **Mass**: kilograms (kg)
- **Force (thrust/drag)**: Newtons (N)
- **Fuel flow**: kg/s
- **Emissions**: g/s
- **Aero/Geo modules**: SI units (meters, m/s, Pa, K, degrees)

All performance classes (`Thrust`, `Drag`, `FuelFlow`, `Emission`) accept an optional `backend` parameter to switch between NumPy (default), CasADi, or JAX computation. See [Backends](backends.qmd) for details.
```

**Step 2: Commit**

```bash
git add api/index.qmd
git commit -m "docs: add API reference intro page"
```

---

### Task 3: Create `api/prop.qmd`

**Files:**
- Create: `api/prop.qmd`

**Content:** Document all 5 public functions in `openap.prop`:
- `available_aircraft(use_synonym=False)`
- `aircraft(ac, use_synonym=False)`
- `aircraft_engine_options(ac)`
- `search_engine(eng)`
- `engine(eng)`

Each function gets: one-line description, parameter table, return description, and a runnable `{python}` example.

**Step 1: Write the file and commit**

```bash
git add api/prop.qmd
git commit -m "docs: add API reference for openap.prop"
```

---

### Task 4: Create `api/thrust.qmd`

**Files:**
- Create: `api/thrust.qmd`

**Content:** Document `Thrust` class:
- Constructor: `Thrust(ac, eng=None, backend=None)`
- Methods: `takeoff(tas, alt=0, dT=0)`, `cruise(tas, alt, dT=0)`, `climb(tas, alt, roc, dT=0)`, `descent_idle(tas, alt, dT=0)`

Each method: description, parameter table with units (kt, ft, fpm, K), returns (N), runnable example.

Note for `descent_idle`: mention the 7% max-thrust approximation.

**Step 1: Write the file and commit**

```bash
git add api/thrust.qmd
git commit -m "docs: add API reference for openap.Thrust"
```

---

### Task 5: Create `api/drag.qmd`

**Files:**
- Create: `api/drag.qmd`

**Content:** Document `Drag` class:
- Constructor: `Drag(ac, wave_drag=False, backend=None)`
- Methods: `clean(mass, tas, alt, vs=0, dT=0)`, `nonclean(mass, tas, alt, flap_angle, vs=0, dT=0, landing_gear=False)`

Parameter tables with units. Note `wave_drag` is experimental.

**Step 1: Write the file and commit**

```bash
git add api/drag.qmd
git commit -m "docs: add API reference for openap.Drag"
```

---

### Task 6: Create `api/fuelflow.qmd`

**Files:**
- Create: `api/fuelflow.qmd`

**Content:** Document `FuelFlow` class:
- Constructor: `FuelFlow(ac, eng=None, backend=None)`
- Methods: `enroute(mass, tas, alt, vs=0, acc=0, dT=0, limit=True)`, `takeoff(tas, alt=None, throttle=1)`, `at_thrust(total_ac_thrust)`, `plot_model(plot=True)`

**Step 1: Write the file and commit**

```bash
git add api/fuelflow.qmd
git commit -m "docs: add API reference for openap.FuelFlow"
```

---

### Task 7: Create `api/emission.qmd`

**Files:**
- Create: `api/emission.qmd`

**Content:** Document `Emission` class:
- Constructor: `Emission(ac, eng=None, backend=None)`
- Methods: `co2(ffac)`, `h2o(ffac)`, `soot(ffac)`, `sox(ffac)`, `nox(ffac, tas, alt=0, dT=0)`, `co(ffac, tas, alt=0, dT=0)`, `hc(ffac, tas, alt=0, dT=0)`

Group into: simple (fuel-proportional: co2, h2o, soot, sox) and altitude-corrected (nox, co, hc).

**Step 1: Write the file and commit**

```bash
git add api/emission.qmd
git commit -m "docs: add API reference for openap.Emission"
```

---

### Task 8: Create `api/wrap.qmd`

**Files:**
- Create: `api/wrap.qmd`

**Content:** Document `WRAP` class with all ~30 methods. Group by flight phase:
- **Takeoff**: `takeoff_speed()`, `takeoff_distance()`, `takeoff_acceleration()`
- **Initial climb**: `initclimb_vcas()`, `initclimb_vs()`
- **Climb**: `climb_range()`, `climb_const_vcas()`, `climb_const_mach()`, `climb_cross_alt_concas()`, `climb_cross_alt_conmach()`, `climb_vs_pre_concas()`, `climb_vs_concas()`, `climb_vs_conmach()`
- **Cruise**: `cruise_range()`, `cruise_alt()`, `cruise_init_alt()`, `cruise_max_alt()`, `cruise_mach()`, `cruise_max_mach()`, `cruise_mean_vcas()`
- **Descent**: `descent_range()`, `descent_const_mach()`, `descent_const_vcas()`, `descent_cross_alt_conmach()`, `descent_cross_alt_concas()`, `descent_vs_conmach()`, `descent_vs_concas()`, `descent_vs_post_concas()`
- **Approach/Landing**: `finalapp_vcas()`, `finalapp_vs()`, `landing_speed()`, `landing_distance()`, `landing_acceleration()`

All methods take no arguments and return dict with `default`, `minimum`, `maximum` (and sometimes `statistic`).

**Step 1: Write the file and commit**

```bash
git add api/wrap.qmd
git commit -m "docs: add API reference for openap.WRAP"
```

---

### Task 9: Create `api/flightphase.qmd`

**Files:**
- Create: `api/flightphase.qmd`

**Content:** Document `FlightPhase` class:
- Constructor: `FlightPhase()`
- Methods: `set_trajectory(ts, alt, spd, roc)`, `phaselabel(twindow=60)`, `flight_phase_indices()`

List the phase labels: GND, CL, DE, CR, LVL.

**Step 1: Write the file and commit**

```bash
git add api/flightphase.qmd
git commit -m "docs: add API reference for openap.FlightPhase"
```

---

### Task 10: Create `api/flightgenerator.qmd`

**Files:**
- Create: `api/flightgenerator.qmd`

**Content:** Document `FlightGenerator` class:
- Constructor: `FlightGenerator(ac, random_seed=42, use_synonym=False)`
- Methods: `enable_noise()`, `climb(**kwargs)`, `descent(**kwargs)`, `cruise(**kwargs)`, `complete(**kwargs)`

Document keyword arguments for each method. Note that return type is `pd.DataFrame`.

**Step 1: Write the file and commit**

```bash
git add api/flightgenerator.qmd
git commit -m "docs: add API reference for openap.FlightGenerator"
```

---

### Task 11: Create `api/aero.qmd`

**Files:**
- Create: `api/aero.qmd`

**Content:** Document module-level constants (kts, ft, fpm, nm, etc.) in a table, then the `Aero` class and all methods:
- Atmosphere: `atmos(h, dT=0)`, `temperature(h, dT=0)`, `pressure(h, dT=0)`, `density(h, dT=0)`, `vsound(h, dT=0)`, `h_isa(p, dT=0)`
- Conversions: `tas2mach`, `mach2tas`, `eas2tas`, `tas2eas`, `cas2tas`, `tas2cas`, `mach2cas`, `cas2mach`, `crossover_alt`

**Important:** Note that aero module uses **SI units** (meters, m/s, Pa, K) — unlike the performance classes which use kt/ft.

Also mention the module-level convenience functions (`openap.aero.cas2tas(...)` etc.).

**Step 1: Write the file and commit**

```bash
git add api/aero.qmd
git commit -m "docs: add API reference for openap.aero"
```

---

### Task 12: Create `api/geo.qmd`

**Files:**
- Create: `api/geo.qmd`

**Content:** Document `Geo` class:
- `distance(lat1, lon1, lat2, lon2, h=0)` — returns meters
- `bearing(lat1, lon1, lat2, lon2)` — returns degrees
- `latlon(lat1, lon1, d, brg, h=0)` — returns (lat, lon)
- `solar_zenith_angle(lat, lon, timestamp)` — returns degrees

Also mention module-level convenience functions.

**Step 1: Write the file and commit**

```bash
git add api/geo.qmd
git commit -m "docs: add API reference for openap.geo"
```

---

### Task 13: Create `api/contrail.qmd`

**Files:**
- Create: `api/contrail.qmd`

**Content:** Document all public functions grouped by topic:
- **Saturation**: `saturation_pressure_over_water(T)`, `saturation_pressure_over_ice(T)`
- **Humidity**: `relative_humidity(q, p, T, to="ice")`, `rhw2rhi(rh_w, T)`
- **Schmidt-Appleman**: `critical_temperature_water(p, eta=0.4)`, `critical_temperature_water_and_ice(p, eta=0.4)`
- **Radiative forcing**: `rf_shortwave(zenith, tau, tau_c)`, `rf_longwave(olr, T)`, `rf_net(zenith, tau, tau_c, olr, T)`
- **Utilities**: `contrail_optical_properties(age_hours)`, `load_olr(filepath, lat, lon, time)`

Also list the module constants (gas constants, emission index, etc.).

**Step 1: Write the file and commit**

```bash
git add api/contrail.qmd
git commit -m "docs: add API reference for openap.contrail"
```

---

### Task 14: Create `api/backends.qmd`

**Files:**
- Create: `api/backends.qmd`

**Content:**
- Explain the backend system: `NumpyBackend` (default), `CasadiBackend`, `JaxBackend`
- Show the `MathBackend` protocol (list all methods in a table)
- Show how to use each backend with a performance class
- Show the convenience modules: `import openap.casadi as oc` and `import openap.jax as oj`
- Note install extras: `pip install openap[casadi]`, `pip install openap[jax]`

**Step 1: Write the file and commit**

```bash
git add api/backends.qmd
git commit -m "docs: add API reference for backends"
```

---

### Task 15: Verify the full book renders

**Step 1: Run quarto preview**

```bash
cd ~/arc/website/openap.dev && quarto render
```

**Step 2: Fix any rendering issues**

Check for broken links, code execution errors, or formatting problems.

**Step 3: Final commit**

```bash
git add -A
git commit -m "docs: complete Part III API Reference"
```
