<br>

# Soil Data Research

<h3>A Global Good Project</h3>

We are gathering data concerning the [Earth System]([https://www.science.org/doi/10.1126/science.1259855](https://education.nationalgeographic.org/resource/earths-systems#geosphere)) and [Planetary Boundary](https://www.science.org/doi/10.1126/science.1259855).

This series of articles explores a path from data collection to visualization and machine learning from the human-planet perspective. Please contribute to helping this project grow.

![Soil and prisma](images/dalle_earth_prisma.png)
Image Created with [DALL·E](https://labs.openai.com/s/MXs5BGiumacsHINDgiS3bOGg)

My background is in operations, people, analysis, and mathematics. Particular interest to me is the soil system's health.

Coincidentally, [Hackers News](https://news.ycombinator.com/item?id=32293359) led me to an article about [Soil.Spectroscopy 4 Global Good](https://soilspectroscopy.org/): open soil [database](https://soilspectroscopy.github.io/ossl-manual/), open source software and more. This database was a great starting point to get familiar with some of the field.

<br>

---

**Table of Contents**

- [Project 1: Load Soil Site Data with Javascript](#project-1-load-soil-site-data-with-javascript)
  - [Install and Run Tests](#install-and-run-tests)
  - [Loader](#loader)
  - [Bin to h3 cells and Convert to GeoJSON](#bin-to-h3-cells-and-convert-to-geojson)
- [Soil Research](#soil-research)
  - [Spectroscopy](#spectroscopy)
  - [Soil](#soil)
    - [Minerals](#minerals)
    - [Organic Matter (SOM)](#organic-matter-som)
    - [Structure](#structure)
    - [Organic Carbon (SOC)](#organic-carbon-soc)
  - [Visual Assessment of Soils](#visual-assessment-of-soils)
  - [Soil Health](#soil-health)
- [The Open Soil Spectral Library (OSSL)](#the-open-soil-spectral-library-ossl)
  - [OSSL Explorer](#ossl-explorer)
  - [OSSL manual](#ossl-manual)
- [Start](#start)
  - [Connecting to the OSSL database](#connecting-to-the-ossl-database)
  - [Exporting a Soil Site Sample from the OSSL as CVS](#exporting-a-soil-site-sample-from-the-ossl-as-cvs)
- [End](#end)
  - [About Us](#about-us)
  - [Updates](#updates)
  - [Contact Us](#contact-us)

---

<br>

# Project 1: Load Soil Site Data with Javascript

Example image generated by a range query to SQLite database. Partitioned into cells by (sum of samples):

![image of soil site data](images/soilsite_d3.png)

## Install and Run Tests

Try out the first project that generates geoJSON from a local SQLite database by installing with `node.js`. From the main directory. This loads, parses, and visualizes the data with a local image `d3.js` globe. The data are partitioned into cells with the [`h3-js`](https://github.com/uber/h3-js) library. For the first step we load only the ID and latitude/longitude of the sample. See `src/js/database.js` and `src/js/globe.js` for more details.

```sh
npm install
npm test
```

## Loader

The core piece of code is loader for OSSL `csv` from the `soilsite` collection. It looks like this:

```js
async function load(dataToLoad) {
  const inputStream = createReadStream(dataToLoad, "utf8");
  const parser = inputStream.pipe(parse());
  const createSql = "CREATE TABLE soilsite (id TEXT, lat REAL, lon REAL)";
  db.run(createSql);
  const insertSql = "INSERT INTO soilsite VALUES (?, ?, ?)";
  const state = db.prepare(insertSql);
  let count = 0;

  // every record
  for await (const record of parser) {
    if (count !== 0) {
      state.run(record[0], record[10], record[11]);
    }
    count++;
  }
  state.finalize();
}
```

## Bin to h3 cells and Convert to GeoJSON

```js
function pointsToH3Cells(points, resolution = 7) {
  const cells = {};
  for (const point in points) {
    const cell = pointToH3Cell(points[point][0], points[point][1], resolution);
    if (cells[cell]) {
      cells[cell]++;
    } else {
      cells[cell] = 1;
    }
  }
  return cells;
}

function cellsToGeoJson(cells) {
  const geojson = h3SetToFeatureCollection(Object.keys(cells), (hex) => ({
    value: cells[hex],
  }));
  return geojson;
}
```

<br>

# Soil Research

## Spectroscopy

The importance of [spectroscopy](https://en.wikipedia.org/wiki/Spectroscopy) is centered around the fact that every element in the periodic table has a unique light spectrum.

[Soil spectroscopy](<https://soilspectroscopy.org/about/#:~:text=Soil%20spectroscopy%20offers%20a%20safe%20and%20rapid%20alternative%20(Figure%201).&text=Soil%20spectroscopy%20is%20the%20measurement,applied%20to%20a%20soil%20surface.>) is the measurement of light absorption when a light in the visible, near-infrared or mid-infrared (Vis–NIR-MIR) regions of the electromagnetic spectrum is applied to a soil surface:

![Spectroradiometer](images/spectroscopy.png)
[Image Source](<https://soilspectroscopy.org/about/#:~:text=Soil%20spectroscopy%20offers%20a%20safe%20and%20rapid%20alternative%20(Figure%201).&text=Soil%20spectroscopy%20is%20the%20measurement,applied%20to%20a%20soil%20surface.>)

The reflected infrared radiation is converted to electrical energy and fed to a computer for interpretation. Each major organic component of the soil absorbs and reflects light differently. By measuring these different reflectance characteristics, the Spectroradiometer and a computer determine the ingredients in the soil sample.

A typical soil spectrum in the (A) visible, (B) near-infrared, and (C) mid-infrared portion of the Electromagnetic Spectrum:

![Explorer](images/spectrum.png)
[Image Source: Advances in Agronomy](https://www.sciencedirect.com/topics/agricultural-and-biological-sciences/reflectance-spectroscopy#:~:text=NEAR%2DINFRARED%20REFLECTANCE%20SPECTROSCOPY%20ANALYSIS,%2C%20energy%2C%20and%20mineral%20content.)

Light absorption in the VIS region is due to the excitation of electrons.
For longer wavelengths, NIR-MIR, the absorption is due to [vibrations](https://youtu.be/QVOx4oFugts) in the chemical bonds within molecules: symmetrical stretch, asymmetrical stretch, and bending vibrations.

The spectra will show overtones and combinations of these vibrations, mainly in the NIR region.

The mathematical modeling of these vibrations, combinations, and overtones can be analyzed using [polynomial algebra](https://pdf.sciencedirectassets.com/272313/1-s2.0-S0747717105X01055/1-s2.0-S0747717105000076/main.pdf?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEHYaCXVzLWVhc3QtMSJIMEYCIQCOdUdiZizjPL3zil542SGm2uyxqyRb6DRKD9sUktpDRgIhAPHMnB7lg7TW7EBDSY04NhAWyJbm8xxB1WssAnb6AR9pKtsECN%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQBRoMMDU5MDAzNTQ2ODY1IgzlDnUTMXwOzbCX9hYqrwRhSBJiiEtARNv%2BtOXGEYlJ8Ct%2FzhBBZ5Ja5oiK71WIiXk%2F66bCCpHkhz510DtFbBIc%2BRKf2k%2Ffk4L9yW89TA6EHWA3F72XdpH38fiCelqvnk6PXuGOHw5e6AtC4lt0e%2BoNGovcxN1wtOOJF8W82ocBYsQfCJVhA0XskbQKZQatJMmUHyI5fwdr6U%2F1V6xnceQHqre5nkECVfIYivqLIXnqs4rz%2FM3PnNyL8Cbk35dhXOXaHQ%2FZTcN0IXemz%2BZPMiPF2ToibD8D4%2B4qSlc%2F%2FkBK%2FfSHqNyUXNAzMrxpErNMfZbkZdoHT9HwWbFFv8dfPPfhSetNmpzuZKcXaHk47X54FoPBbWexrdxY3yXiSzmQU6m5uwq6jzjVyza5u3gRZFeb4d6ysGpHwwI3fAwOfJQ%2FBHSrIa9vBFH%2BQB0fhkDpoGiZXImhihgM6RbdlSZd6uUvIvFOqNy0rrHS0AGbWGuKdDKNGBVbae4qEM2I2dgpT4ssSGCwGnv24f5wBh7r0QZI2c3REuZJ3oSnVyw104K5rgXdz3FuelTagRR8a2mV4Wc%2BbvUvLJ%2Fk9OXLFhNZp2uusjNt%2B%2Bb7AA8Mdq0nyax7TdXdpqJyAz%2B7PyENIutUl%2F5mpc%2FnETbhUwIoLyXPGxvAtC0ji27Xz87d1uvzGts1ms4TXK57VimY68MhQEDsMrwviw7IKr1EMEc7zmi6wy8aiQqTuCNfH%2FSjLdYgYBB2IvJofNnFGHUpdkYvsqIZMJCa8JcGOqgBSAWPtfsSUVUMA2IdSPMt7EHFX6XwUXgGgRVnd77ElzjFOHuyPN3iCl6CGTsR5MDMNCs%2BiZwqnzUgkY8mSMqdvnX7JqajkWYTYAdjZOt5QnFCKj7vsXqij9X35Mkq6u0QHwwEyrpst4LVuwrY0cao6CSLWUUNm%2FxfuzzxFwQVhJ2bZ5swOrF9TqO1aVZMHMK8Vzl8lxnFwAwwU6TUZ9XwOIR1brGJhZpa&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220816T224755Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAQ3PHCVTYXRGOHNQV%2F20220816%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=442f2292030185d7ae54065a2d79193cb6c1386941d947989ed83753a14a624e&hash=75ea736321900bd2518869cf80773aace3203dbff5b41f155cffea9a364dec99&host=68042c943591013ac2b2430a89b270f6af2c76d8dfd086a07176afe7c76c2c61&pii=S0747717105000076&tid=spdf-d39efd61-93b1-481e-8092-2fa06b42003f&sid=e9f2334a127c1247f66a6e45c2354cf521cegxrqa&type=client&ua=5155030a045707035109&rr=73bdb6ee4f772d58) as tools or functions:

![Overtone_polynomials](images/overtone_table.png)
[Image Source](https://www.youtube.com/watch?v=QVOx4oFugts)

Water has unique soil spectral features (Absorbance vs. Wavelength):

![Spectra_overtones_water](images/overtone_spectra.png)
[Image Source](https://www.youtube.com/watch?v=QVOx4oFugts)

## Soil

Soil is a living system working as a life-sustaining resource. It teams up with billions of bacteria, fungi, and other microbes to create an abundant [soil community](https://www.sciencedirect.com/topics/earth-and-planetary-sciences/microbial-community) filled with diverse soil biota.

Soils have [4 essential components](http://www.nzdl.org/cgi-bin/library?e=d-00000-00---off-0hdl--00-0----0-10-0---0---0direct-10---4-------0-1l--11-en-50---20-about---00-0-1-00-0-0-11----0-0-&a=d&c=hdl&cl=CL1.16&d=HASH412cd503b5262205ac14c6.3.1):

- Mineral particles: sand, silt, and clay
- Organic matter
- Water
- Air

Organism abundance, diversity, and activity are not randomly distributed in the soil but vary in a patchy fashion both horizontally across a landscape and vertically through the [soil profile](https://www.sciencedirect.com/topics/earth-and-planetary-sciences/soil-biota):

![Soil_horizons](images/soil_horizons_screens.png)
[Image Source: Soil Horizons](https://www.youtube.com/watch?v=_aZbGBaP_7Y)

Most soils evolve slowly over centuries through the weathering of underlying rocks and the decomposition of organic matter. Other soils are formed from deposits laid down by rivers, seas, or wind forces.

> A sample of typical [topsoil](http://www.nzdl.org/cgi-bin/library?e=d-00000-00---off-0hdl--00-0----0-10-0---0---0direct-10---4-------0-1l--11-en-50---20-about---00-0-1-00-0-0-11----0-0-&a=d&c=hdl&cl=CL1.16&d=HASH412cd503b5262205ac14c6.3.1) contains about
>
> - ~50% pore space filled with varying proportions of air and water,
> - ~50% of mineral particles and organic matter
>
> Contrary to 30-100% organic matter found in marshes, bogs, and swamps soils.

### Minerals

Soil minerals give soil different texture attributes and colors. Minerals are classified by size.

  <table>
  <th colspan="5">Minerals Classified by Size</th> 
  <tr>
  <th>Type</th>
  <th>Size (mm)</th>
  <th>Texture</th>
  <th>Characteristics</th>
  <th></th>
  </tr>
  <tr>
  <td>Sand</td>
  <td>2.0 - 0.05</td>
  <td>Gritty</td>
  <td>Quite visible, consists of small particles with low surface area, significant drainage</td>
  <td rowspan="3">   <img src="images/mineral_size.png" alt="Soil Minerals" width="90%">
  <a href="https://www.nature.com/scitable/knowledge/library/what-are-soils-67647639/">
  Image Source</a>
  </a>
  </td>
  </tr>
  <tr>
  <td>Silt</td>
  <td>0.05 - 0.002</td>
  <td>Buttery</td>
  <td>Not visible, increases the water holding capacity</td>
  </tr>
  <tr>
  <td>Clay</td>
  <td>< 0.002</td>
  <td>Sticky</td>
  <td>High water holding capacity, smallest pores, and large charged surfaces attract and retain nutrients.</td>
  </tr>
  <!-- add reference link to table footer -->
  <tfoot>
  <tr>
  <td colspan="5">
  <a href="https://www.ctahr.hawaii.edu/mauisoil/a_comp01.aspx">
  Table Source
  </a>
  </td>
  </tr>
  </tfoot>
  </table>

The most common mineral in soils is [quartz](http://www.nzdl.org/cgi-bin/library?e=d-00000-00---off-0hdl--00-0----0-10-0---0---0direct-10---4-------0-1l--11-en-50---20-about---00-0-1-00-0-0-11----0-0-&a=d&c=hdl&cl=CL1.16&d=HASH412cd503b5262205ac14c6.3.4); it is not very reactive. But on the other hand, clay is very reactive.
Clay particles can form strongly protected structures that [store soil C](https://www.sciencedirect.com/topics/earth-and-planetary-sciences/soil-aggregate) for long periods.

These protected structures made with clay ensure good water-holding capacity and provide a good [source of plant nutrients](http://www.nzdl.org/cgi-bin/library?e=d-00000-00---off-0hdl--00-0----0-10-0---0---0direct-10---4-------0-1l--11-en-50---20-about---00-0-1-00-0-0-11----0-0-&a=d&c=hdl&cl=CL1.16&d=HASH412cd503b5262205ac14c6.3.4).

### Organic Matter (SOM)

Soil organic matter [SOM](<https://www.agric.wa.gov.au/measuring-and-assessing-soils/what-soil-organic-carbon#:~:text=Soil%20organic%20carbon%20(SOC)%20refers,to%20measure%20and%20report%20SOC.>) is composed mainly of carbon, hydrogen, and oxygen. In addition, it contains small amounts of other elements, such as nitrogen, phosphorous, sulfur, potassium, calcium, and magnesium, in organic residues. It is divided into 'living' and 'dead' components and can range from very recent inputs, such as stubble, to decayed materials that might be many hundreds of years old. About 10% of below-ground SOM, such as roots, fauna, and microorganisms, is living.

SOM exists as four distinct fractions which vary widely in size, turnover time, and composition in the soil:

- dissolved organic matter
- particulate organic matter
- stable organic matter or humus
- resistant organic matter

![SOM](images/som.png)
[Image Source](<https://www.agric.wa.gov.au/measuring-and-assessing-soils/what-soil-organic-carbon#:~:text=Soil%20organic%20carbon%20(SOC)%20refers,to%20measure%20and%20report%20SOC.>)

### Structure

[Soil structure](https://www.sciencedirect.com/topics/earth-and-planetary-sciences/soil-structure) refers to the proportions of solids and voids. A key aspect of soil structure is the aggregation of individual mineral and organic particles into larger units.

Aggregates are separated into size classes: macroaggregates (250 μm–2 mm) and microaggregates (53–250 μm).

Macroaggregates are formed when fungi and bacteria decompose fresh plant residue or, technically speaking, [light fraction SOM](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0033217#:~:text=The%20light%20fraction%20is%20a,inputs%20and%20decomposition%20%5B31%5D.).

Bacterial secretion of high-molecular-weight sugar-based polymers (EPSs). These EPSs and fungal hyphae serve as nucleation cores to accrete larger masses of slightly decayed SOM that become macroaggregates. These macroaggregates are constantly weathering in the soil to produce microaggregates within SOM:

![Macro_micro_aggregates](images/soil_matrix.png)

![Macro_micro_aggregates](images/water_gas.png)
[Image Source: American Society of Microbiology](https://journals.asm.org/doi/10.1128/AEM.00324-19#)

### Organic Carbon (SOC)

Soil Organic Carbon [SOC](<https://www.agric.wa.gov.au/measuring-and-assessing-soils/what-soil-organic-carbon#:~:text=Soil%20organic%20carbon%20(SOC)%20refers,to%20measure%20and%20report%20SOC.>) refers to the carbon components in organic compounds. Soil organic matter (SOM) is challenging to measure directly, so laboratories tend to measure and report SOC. Soil organic carbon is a measurable component of soil organic matter which contributes to nutrient retention and turnover, soil structure, moisture retention and availability, degradation of pollutants, and carbon sequestration. SOC has been identified as a [global indicator for monitoring soil health and productivity](https://www.openaccessgovernment.org/what-is-soil-organic-carbon-soc/120702/).

## Visual Assessment of Soils

Soil color is usually due to 3 primary pigments:

- black—from organic matter
- red—from iron and aluminum oxides
- white—from silicates and salt.

![Soil_color](images/sample_soils.png)

![Soil_color](images/sample_pigment.png)
[Image Source](https://www.nrcs.usda.gov/wps/portal/nrcs/detail/wi/soils/?cid=NRCSEPRD1370419#:~:text=Color%20development%20and%20distribution%20of,manganese%20forms%20black%20mineral%20deposits.)

Soil texture can also be asses by estimating the size of the observable particle:

![Sample_triangle](images/sample_triangle.png)
[Image Source](https://gardentutor.com/lessons/soil/)

Soil color and observable soil texture are valuable indicators of the chemical processes beneath the surface and can offer some quick soil management information:

![Sample_full](images/sample_full.png)
<br>
[Image Source: Queensland Government](https://www.qld.gov.au/environment/land/management/soil/soil-properties/colour)

## Soil Health

The basic principles of soil health:

<table>
  <tr>
    <td>
      <ul>
        <li><h3><b> ⇑ Presence Living Roots</b></h3></li>
        <li><h3><b> ⇑ Soil Cover</b></h3></li>
        <li><h3><b> ⇑ Biodiversity</b></h3></li>
        <li><h3><b> ⇓ Disturbance</b></h3></li>
      </ul>
    </td>
    <td>
      <img src="images/soil_health.png" alt="Soil Health">
      <a href="https://www.nrcs.usda.gov/wps/portal/nrcs/main/soils/health/"> Image Source: USDA</a>
    </td>
  </tr>
</table>

# The Open Soil Spectral Library (OSSL)

The [Open Soil Spectral Library (OSSL)](https://soilspectroscopy.github.io/ossl-manual/) is a global good project which serves collections of soil properties derived from spectral data. OSSL is also a network that delivers robust statistical models, calibration and prediction models, research tools, and opportunities to collaborate across borders.

The initiative received a funding award through the National Institute of Food and Agriculture (USDA).
[NIFA](https://www.nifa.usda.gov/about-nifa/press-releases/nifa-invests-over-7-million-big-data-artificial-intelligence-other) has invested over $7 Million in Big Data, Artificial Intelligence, and Other Cyberinformatics Research.

Among other valuable resources, the OSSL project offers beautifully developed software:

## [OSSL Explorer](https://explorer.soilspectroscopy.org/)

![Explorer](images/ossl_explorer.png)

And the user manual, which is open for contributions:

## [OSSL manual](https://soilspectroscopy.github.io/ossl-manual/)

<br>

# Start

## Connecting to the OSSL database

The OSSL manual mentioned two ways to access the data. The first method uses MongoDb via R; however, the last yields a certification error. See the image below:

![cert_error](images/cert_error.png)

As an alternative, we tried to connect directly with Javascript through NodeJS, but we also ran into another certificate error.

```
/Users/dev/code/soil_data_research/node_modules/mongodb/lib/utils.js:419
                    throw error;
                    ^

MongoServerSelectionError: certificate has expired
    at Timeout._onTimeout (/Users/dev/code/soil_data_research/node_modules/mongodb/lib/sdam/topology.js:293:38)
    at listOnTimeout (node:internal/timers:564:17)
    at process.processTimers (node:internal/timers:507:7) {
  reason: TopologyDescription {
    type: 'Unknown',
```

Lastly, we used the second method from the OSSL manual to access the data with Studio 3T and inserted the following parameters:

- Connection Name: `soilspec4gg`
- Server: `api.soilspectroscopy.org`
- Authentication DB: `soilspec4gg`
- User name: `soilspec4gg`
- Password: `soilspec4gg`
- Use SSL: `true`
- Accept any SSL certificates: `true`

To see full details of this step go to: [OSSL connect](step_studio3T.md).

## Exporting a Soil Site Sample from the OSSL as CVS

To see full details of this step go to: [database download](exporting_OSSL.md).

![OSSL_download](/images/export_sample_screen3.png)

# End

<br>

## **About Us**

Highlights of my background:

- [Lifelong learner](https://en.wikipedia.org/wiki/Lifelong_learning) with a background in operations, people, analysis, and mathematics.
- Active Software Engineering and Open Source Technology: JS, Python, HTML, CSS, automated testing, and continuous integrations.
- Currently developing with a [team](https://chromatic.systems) software about Earth System's Health.
- Growing experiment +25 Organic Plants: herbs, tree seedlings, vegetables, fruits, flowers, and fungi; in a ~1 $m^2$ box.
- Creation of +400 Plant-Based Sense Immersions Healing Meal.
- Formerly, Goldman Sachs, Copa Airlines, GDI 2021 Hackaton, and a researcher at the University of Texas, Systems.

## **Updates**

<ul>
<li>I am available for contract work or full-time employment. </li>

<li>I am collaborating with a team: <a href="https://chromatic.systems">chromatic.systems. </a>

</li>

<li>Here is a link to this project's
<a href="https://github.com/mejia-code/soil_data_research.git"> code.</a>

</li>

<li>The following post will explore a simple Machine Learning model.</li>
</ul>
 <br>
Thank you for reading!
<br>
<br>

## **Contact Us**

- email: code@patimejia.com
- websites:
  - [pati](https://patimejia.com)
  - [chromatic.systems](https://chromatic.systems)
    <br>
