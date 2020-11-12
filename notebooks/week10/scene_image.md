---
jupytext:
  notebook_metadata_filter: all,-language_info,-toc,-latex_envs
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.12
    jupytext_version: 1.6.0
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

```{code-cell} ipython3
import a301_lib
import rasterio
from pathlib import Path
from matplotlib import pyplot as plt
import seaborn as sns
from skimage import img_as_ubyte
from skimage import exposure
import numpy as np
import PIL
```

```{code-cell} ipython3
notebook_dir=Path().resolve().parent
print(notebook_dir)
week10_scene = notebook_dir / "week10/vancouver_345_refl.tiff"

with rasterio.open(week10_scene) as van_raster:
    b3_refl = van_raster.read(1)
    chan1_tags=van_raster.tags(1)
    b4_refl = van_raster.read(2)
    chan2_tags=van_raster.tags(2)
    b5_refl = van_raster.read(3)
    chan3_tags=van_raster.tags(3)
    crs = van_raster.profile['crs']
    transform = van_raster.profile['transform']
    tags=van_raster.tags()
print(tags,chan1_tags)
```

```{code-cell} ipython3
plt.imshow(b3_refl);
```

```{code-cell} ipython3
sns.jointplot(x=b3_refl.flat, y=b4_refl.flat,xlim=(0,0.2),ylim=(0.,0.2),
              kind="hex", color="#4CB391")
```

```{code-cell} ipython3
sns.jointplot(x=b4_refl.flat, y=b5_refl.flat, kind="hex" ,xlim=(0,0.3),ylim=(0.,0.5),
              color="#4CB391")
```

```{code-cell} ipython3
channels = np.empty([3, b3_refl.shape[0], b3_refl.shape[1]], dtype=np.uint8)
for index, image in enumerate([b3_refl, b4_refl, b5_refl]):
    stretched = exposure.equalize_hist(image)
    channels[index, :, :] = img_as_ubyte(stretched)
```

```{code-cell} ipython3
plt.imshow(channels[0,:,:]);
```

```{code-cell} ipython3
sns.jointplot(x=channels[0,...].flat, y=channels[1,...].flat,xlim=(0,255),ylim=(0.,255),
              kind="hex", color="#4CB391");
```

```{code-cell} ipython3
png_filename = notebook_dir / "week10/vancouver_345_stretched.png"
num_chans, height, width = channels.shape
with rasterio.open(
    png_filename,
    "w",
    driver="PNG",
    height=height,
    width=width,
    count=num_chans,
    dtype=channels.dtype,
    crs=crs,
    transform=transform,
    nodata=0.0,
) as dst:
    chan_tags = ["LC8_Band3_refl_counts", "LC8_Band4_refl_counts", "LC8_Band5_refl_counts"]
    dst.update_tags(**tags)
    dst.update_tags(written_on=str(datetime.date.today())
    dst.update_tags(history="written by scene_image.md")
    for index, chan_name in enumerate(chan_tags):
        dst.update_tags(index + 1, name=chan_name)
        dst.update_tags(index + 1, valid_range="0,1")
    dst.write(channels)
    keys = ["3", "4", "5"]
    for index, chan_name in enumerate(keys):
        dst.update_tags(index + 1, name=chan_name)
```

```{code-cell} ipython3
from IPython.display import Image
Image(filename=png_filename,width="80%") 
```