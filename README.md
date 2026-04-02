# linlinhua.github.io

Personal portfolio of Lin-Lin Hua — GPU Systems Engineer & Researcher.

## Structure

```
index.html                  ← Main portfolio (single-page app)
.nojekyll                   ← Tells GitHub Pages to skip Jekyll
assets/
  img/posts/                ← Project images & screenshots
    linlinhua.JPG
    nsight_gpu_throughput.png
    nsight_roofline.png
    nsight_memory.png
    nsight_warp.png
    nsight_occupancy.png
    wgan_structure.png
    wgan_distribution.png
  pdf/                      ← PDFs (resume, papers, proposals)
    LinLin_Hua_Resume.pdf
    MOR-GANs_Paper.pdf
    EE541_Intent_Classification_Proposal.pdf
    EE547_Proposal.pdf
```

## Deployment

Push to `main` — GitHub Pages deploys automatically.

No build step required. The site is a single `index.html` with all styles and JS inline.
Images are referenced as relative paths from `assets/`.

## Adding a new project

1. Add a new `<div id="proj-yourname">` block inside `#work` in `index.html`
2. Add a row to the home project list and the work-list
3. Add any images to `assets/img/posts/`
