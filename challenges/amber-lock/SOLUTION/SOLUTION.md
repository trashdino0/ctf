> # ⚠️ SPOILER — FULL SOLUTION
>
> **Everything below gives away every answer.** Do not open this file until you
> have finished attempting the challenge, or you will spoil it for yourself.
>
> Want a nudge instead? Use [`../hints/HINTS.md`](../hints/HINTS.md).

---

# Amber Lock — solution

**Flag:** `CTF{4mb3r_cl4d3_1nd3x}`

The challenge is a straight knowledge ladder. There is no hidden code in the
page: the JavaScript only checks answers, reveals fragments, and assembles the
string. Research is the intended solve path.

## 1. How the fragmentation works

The twelve fragments are handed out in a deliberately scrambled order. Each
fragment carries an `IDX` code that is its true position in the final flag.

| Lock | Correct answer | Fragment received | True index |
| ---: | --- | --- | ---: |
| 1 | Triceratops | `3_` | 09 |
| 2 | Archaeopteryx | `TF` | 02 |
| 3 | Allosaurus | `x}` | 12 |
| 4 | Huincul (Formation) | `b3` | 05 |
| 5 | Mosasaurus | `C` | 01 |
| 6 | Osborn (Henry Fairfield Osborn) | `4d` | 08 |
| 7 | Quill knobs / ulnar papillae | `4m` | 04 |
| 8 | Maniraptora | `d3` | 11 |
| 9 | Epidexipteryx | `r_` | 06 |
| 10 | Jehol (Biota) | `{` | 03 |
| 11 | Nomen dubium | `1n` | 10 |
| 12 | Chilesaurus | `cl` | 07 |

Sorting by the index column produces:

```text
01 C   02 TF   03 {   04 4m   05 b3   06 r_
07 cl  08 4d   09 3_   10 1n   11 d3   12 x}
```

Concatenated:

```text
CTF{4mb3r_cl4d3_1nd3x}
```

Reading the leetspeak as "amber clade index" is a sanity check on the ordering,
but the index codes are authoritative.

## 2. Why each answer is correct

1. **Triceratops** — *Triceratops horridus*, Marsh 1889, Hell Creek Formation,
   Late Maastrichtian. A ceratopsian that coexisted with *T. rex*, not with humans.
2. **Archaeopteryx** — *Archaeopteryx lithographica*, named by Hermann von Meyer
   in 1861 from an isolated Solnhofen feather. A mosaic of avian and theropod
   traits rather than a clean "first bird."
3. **Allosaurus** — *Allosaurus fragilis*, Marsh 1877, Morrison Formation. Greek
   *allos* ("different/strange") + *sauros* ("lizard"), chosen because the
   vertebrae looked unusual.
4. **Huincul Formation** — the Neuquén Formation that yielded both
   *Argentinosaurus huinculensis* and *Mapusaurus*.
5. **Mosasaurus** — *Mosasaurus hoffmannii*, a Late Cretaceous marine squamate.
   Mosasaurs are lizards' relatives, not dinosaurs.
6. **Osborn** — Henry Fairfield Osborn named *Tyrannosaurus rex* in 1905 from
   material collected by Barnum Brown.
7. **Quill knobs** — ulnar papillae on *Velociraptor*'s ulna, reported by Turner,
   Makovicky & Norell in *Science* (2007), demonstrating large pennaceous
   feathers. The scaly movie raptors are outdated.
8. **Maniraptora** — the coelurosaurian clade containing dromaeosaurs,
   troodontids, and birds. Birds are living dinosaurs, not former dinosaurs.
9. **Epidexipteryx** — *Epidexipteryx hui*, a tiny Late Jurassic Chinese
   scansoriopterygid whose name means "Hu's display feather."
10. **Jehol Biota** — the Yixian/Jiufotang Lagerstätte of Liaoning, preserving
    *Sinosauropteryx*, *Confuciusornis*, and many feathered taxa.
11. **Nomen dubium** — Latin for "dubious name," applied when a holotype lacks
    unique diagnostic features.
12. **Chilesaurus** — *Chilesaurus diegosuarezi*, described 2015; the species
    honours Diego Suárez, who found the fossils at age seven. Its chimeric mix of
    traits keeps its phylogenetic position contested.

## 3. Sources used for verification

- [Canadian Museum of Nature — Never Enough Triceratops](https://nature.ca/en/never-enough-triceratops)
- [PNAS — Tyrannosaurus/Triceratops evolution in Hell Creek](https://pmc.ncbi.nlm.nih.gov/articles/PMC4104892)
- [EBSCO — Archaeopteryx lithographica is discovered](https://www.ebsco.com/research-starters/history/archaeopteryx-lithographica-discovered/)
- [Yale Peabody — Allosaurus fragilis, Marsh 1877](http://peabody.yale.edu/store/allosaurus-fragilis-marsh-1877)
- [Smithsonian — Allosaurus type specimen](https://smithsonianmag.com/blogs/national-museum-of-natural-history/2024/04/17/just-our-type-museums-dinosaur-skeleton-becomes-the-scientific-standard)
- [Geological Magazine — rediagnosis of Mosasaurus hoffmannii](https://doi.org/10.1017/s0016756816000236)
- [Smithsonian — The Nation's T. rex](https://naturalhistory.si.edu/explore/dinosaurs-fossils/nations-t-rex)
- [PubMed — Feather quill knobs in the dinosaur Velociraptor](https://pubmed.ncbi.nlm.nih.gov/17885130)
- [UCMP — Maniraptora](https://ucmp.berkeley.edu/diapsids/saurischia/maniraptora.html)
- [Beijing Review — Filling a Missing Link (Epidexipteryx)](http://bjreview.com/print/txt/2008-11/11/content_163343.htm)
- [Oxford NSR — The Jehol Biota](https://academic.oup.com/nsr/article/1/4/543/1508504)
- [Natural History Museum — What was Chilesaurus diegosuarezi?](https://nhm.ac.uk/discover/what-was-chilesaurus-diegosuarezi.html)
