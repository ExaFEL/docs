---
layout: "page"
title: "Spot discovery and refinement"
permalink: disc-refine
---

Continuing with the processing steps discussed in the [detector calibration steps](cspad-calib), we now examine indexing and spot discovery.  At this stage in the processing pipeline our goal is to find Bragg peaks, index them, refine them and perform integration over the data (if necessary).

Ideally, we aim to discover the reciprocal lattice vectors of the crystal lattice unit cell ($$I_{hkl}$$) and learn the structure of the resulting crystals. This is challenging, as the data is collected in single shots at undetermined crystal orientations, and samples only partial lattice sites. By examining the collected data, an estimate of the crystal structure is to be determined and used to better refine the data analysis. For an estimated crystal with specific lattice periodicity, estimated crystal sites can be determined, allowing the data to be collected from within these regions. This allows for a lowering of overall noise in the collected data, as expected reflection sites (though very weak at higher orders) can contribute to the collection and analysis. (see DOI: 10.7554/eLife.05421)
