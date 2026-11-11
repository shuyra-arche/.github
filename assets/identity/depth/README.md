# DEPTH identity assets

These are the **original SVGs from the supplied DEPTH v1 asset package**, retained as vector sources for Shuyra's GitHub presence.

| Location | Use |
| --- | --- |
| [Black lockup](svg/shuyra-lockup-black.svg) | Wide emblem and outlined names, for light backgrounds |
| [White lockup](svg/shuyra-lockup-white.svg) | Wide emblem and outlined names, for dark backgrounds |
| [Black mark](svg/shuyra-depth-black.svg) | Standalone transparent emblem, for light backgrounds |
| [White mark](svg/shuyra-depth-white.svg) | Standalone transparent emblem, for dark backgrounds |
| [Dark avatar](github/shuyra-avatar-dark.svg) | White mark on a dark square |
| [Light avatar](github/shuyra-avatar-light.svg) | Dark mark on a light square |
| [Favicon master](web/favicon.svg) | Dark compact icon; not connected to any website here |

The organization [English](../../../profile/README.md) and [Chinese](../../../profile/README.zh-CN.md) profiles use the **720 × 280 horizontal lockup** (emblem, Shuyra, and 圜 / Σχύρα), switching between black and white artwork with the viewer's color scheme. Standalone marks are reserved for avatars, favicons, and contexts where the wordmark cannot be read.

## Geometry and handling

The artwork was **reconstructed from an approved preview**, not recovered from the original source file. The 440 × 440 master uses an outer circle centered at (220, 220) with radius 140, nested circles with radii 70 and 35, and a stroke width of 10. Keep its geometry, proportions, orientation, and crossings unchanged. Use the existing monochrome colors (`#101010` / `#f8f8f8`) rather than adding gradients or shadows.

The source package also contains raster exports and small-size optical-correction candidates. Those variants are not copied into this repository. The two lockup SVGs preserve the supplied outlined lettering; no installed fonts are needed. At 16–32 px, the mark's dense intersections still need optical review; the favicon here is a source reference, **not a claim that the smallest sizes have been finalized**.

The SVG avatars in this folder are prepared for selection, but storing them in Git does **not** change the organization's GitHub avatar. For that setting, use the 512 px PNG avatar export from the supplied asset package.

No font binaries are included.
