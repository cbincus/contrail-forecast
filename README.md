# Atmospheric cross-sections

Standalone, browser-based tools for visualising numerical weather prediction output at a single point. Each tool is one self-contained HTML file: no build step, no server-side code and no dependencies. Open it in a browser and it fetches the data directly from the [Open-Meteo](https://open-meteo.com/) API.

## Tools

| File | Model | What it shows |
|---|---|---|
| [`icon_cloud_cover_contrails_logp.html`](icon_cloud_cover_contrails_logp.html) | DWD ICON global | Log-pressure / time cross-section of parameterised cloud cover with contrail-formation and contrail-persistence conditions |

More tools may be added over time; each will get its own section below.

---

## ICON cloud cover & contrail cross-section

`icon_cloud_cover_contrails_logp.html`

A time–pressure cross-section (1000–150 hPa, logarithmic pressure axis) for a chosen point, built from DWD ICON global forecasts. It shows:

- **Cloud cover** as grey shading (white = clear, grey = 100 %), parameterised from temperature and relative humidity at each pressure level.
- **Contrail conditions** as a coloured overlay:
  - light blue: contrails form but are short-lived (T < T<sub>c</sub>);
  - violet: persistent contrails (T < T<sub>c</sub> and RH over ice ≥ threshold).
- **Summary bars** above the plot:
  - *Contrails*: the most significant contrail condition at each hour between 150 and 500 hPa;
  - *Mid cloud* and *Low cloud*: the model's own mid- and low-level cloud cover.
- **Ground masking**: levels with pressure higher than the model surface pressure are masked.
- **Flight levels** on the right axis (ICAO standard atmosphere), a "now" marker, and a detailed tooltip with T, RH<sub>w</sub>, vapour pressure, RH<sub>i</sub>, RH for the cloud phase, cloud cover, T<sub>c</sub> and T − T<sub>c</sub> for the nearest model level.

### Usage

Open the file in any modern browser, either locally or via GitHub Pages. Settings:

| Setting | Default | Notes |
|---|---|---|
| Latitude, longitude | 47.0° N, 28.9° E | The nearest ICON grid point is used |
| Past days | 2 | 0–92 |
| Forecast days | 5 | 1–7 |
| Engine contrail factor | 0.0390 g kg⁻¹ K⁻¹ (high-bypass) | Presets 0.0300 / 0.0336 / 0.0340 / 0.0390, or any value in between |
| Persistent if RH<sub>i</sub> ≥ | 90 % | |
| Shading | Blocks (raw levels) | Or *Smooth*, interpolated in time and ln p |

The settings can also be passed in the URL so a view can be bookmarked or shared:

```
icon_cloud_cover_contrails_logp.html?lat=47.0&lon=28.9&past=2&days=5&cf=0.0390
```

### Methods

**Humidity.** ICON reports relative humidity over water (RH<sub>w</sub>). Vapour pressure e = RH<sub>w</sub> · e<sub>sat,w</sub>(T), and RH over ice RH<sub>i</sub> = e / e<sub>sat,i</sub>(T). Saturation vapour pressures use the Tetens formula

e<sub>sat</sub>(T) = a<sub>1</sub> exp[a<sub>3</sub>(T − T<sub>0</sub>) / (T − a<sub>4</sub>)], T<sub>0</sub> = 273.16 K,

with a<sub>1</sub> = 611.21 Pa, a<sub>3</sub> = 17.502, a<sub>4</sub> = 32.19 K over water (Buck, 1981) and a<sub>1</sub> = 611.21 Pa, a<sub>3</sub> = 22.587, a<sub>4</sub> = −0.7 K over ice (Alduchov & Eskridge, 1996).

**Cloud cover** follows Sundqvist et al. (1989):

CC = 1 − √[(1 − RH<sub>phase</sub>) / (1 − RH<sub>crit</sub>)], RH<sub>crit</sub> = 80 %,

with CC = 0 for RH<sub>phase</sub> ≤ 80 % and CC = 100 % for RH<sub>phase</sub> ≥ 100 %. RH<sub>phase</sub> = e / e<sub>sat</sub>, where e<sub>sat</sub> follows the cloud phase: ice for T ≤ 250.16 K, water for T ≥ 273.16 K, and α e<sub>sat,w</sub> + (1 − α) e<sub>sat,i</sub> in between, with α = ((T − 250.16 K) / 23 K)².

**Contrail formation.** The critical temperature T<sub>c</sub> is taken from the tables in the appendix of Schrader (1997), for contrail factors 0.0300, 0.0336, 0.0340 and 0.0390 g kg⁻¹ K⁻¹, and looked up with RH<sub>w</sub>. The tables are interpolated linearly

- in ln p between pressure rows (extrapolated in ln p below 500 hPa, where the tables end),
- in √(1 − RH<sub>w</sub>) between humidity columns, which follows the sharp change of T<sub>c</sub> near saturation,
- in ln CF between contrail factors.

Contrails form where T < T<sub>c</sub>, and are considered persistent where, in addition, RH<sub>i</sub> reaches the selected threshold.

### Data

The page requests hourly temperature and relative humidity on 15 pressure levels (1000, 975, 950, 925, 900, 850, 800, 700, 600, 500, 400, 300, 250, 200 and 150 hPa), plus `cloud_cover_mid`, `cloud_cover_low` and `surface_pressure`, with `models=dwd_icon_global`. The Open-Meteo API is free for non-commercial use and needs no API key.

### Limitations

- Upper-level humidity is one of the less reliable model fields, and forecasts of contrail persistence lose skill after a few days.
- Past days are earlier model output, not observations.
- RH is reported in whole percent, so RH<sub>i</sub> near ice saturation can flicker around the persistence threshold.
- Below 500 hPa T<sub>c</sub> is extrapolated from the tables (about 0.4 K too cold at 1000 hPa); this rarely matters, since such low levels are almost never cold enough for contrails.

## References

- Alduchov, O. A., and R. E. Eskridge, 1996: Improved Magnus form approximation of saturation vapor pressure. *J. Appl. Meteor.*, **35**, 601–609.
- Buck, A. L., 1981: New equations for computing vapor pressure and enhancement factor. *J. Appl. Meteor.*, **20**, 1527–1532.
- Schrader, M. L., 1997: Calculations of aircraft contrail formation critical temperatures. *J. Appl. Meteor.*, **36**, 1725–1729.
- Sundqvist, H., E. Berge, and J. E. Kristjánsson, 1989: Condensation and cloud parameterization studies with a mesoscale numerical weather prediction model. *Mon. Wea. Rev.*, **117**, 1641–1657.
- Zippenfenig, P., 2023: Open-Meteo.com Weather API [Computer software]. Zenodo. https://doi.org/10.5281/ZENODO.7970649

## Data attribution

[Weather data by Open-Meteo.com](https://open-meteo.com/), licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). ICON model data: [Deutscher Wetterdienst (DWD)](https://www.dwd.de/).

## License

<!-- Choose a licence for the code (e.g. MIT) and add a LICENSE file. -->
