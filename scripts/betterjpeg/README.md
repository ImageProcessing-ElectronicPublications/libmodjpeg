# betterjpeg

A bash script using graphicsmagick and modjpeg for modify jpeg files.

```bash
betterjpeg.sh lena.jpg lena.jpg.png lena.jpg.png.jpg
gm compare -highlight-style assign -highlight-color lime -file lena.jpg.diff.jpg lena.jpg lena.jpg.png.jpg 
```

Original | Modify | Composed | Difference
---------|---------|----------|-----------
![Original](https://raw.githubusercontent.com/ImageProcessing-ElectronicPublications/libmodjpeg-samples/main/images/lena.jpg)|![Modify](https://raw.githubusercontent.com/ImageProcessing-ElectronicPublications/libmodjpeg-samples/main/images/lena.jpg.png)|![Result](https://raw.githubusercontent.com/ImageProcessing-ElectronicPublications/libmodjpeg-samples/main/images/lena.jpg.png.jpg)|![Overlay](https://raw.githubusercontent.com/ImageProcessing-ElectronicPublications/libmodjpeg-samples/main/images/lena.jpg.diff.jpg)

See sample images in [libmodjpeg-samples](https://github.com/ImageProcessing-ElectronicPublications/libmodjpeg-samples).

## URL

[libmodjpeg](https://github.com/zvezdochiot/libmodjpeg)

## License

libmodjpeg is released under the [MIT license](../../LICENSE).


## Acknowledgement

Made with :pizza: and :beers: in Switzerland.

This software is based in part on the work of the Independent JPEG Group.

PNG support is provided by [libpng](http://www.libpng.org/pub/png/libpng.html)


## References

[1] R. Jonsson, "Efficient DCT Domain Implementation of Picture Masking and Composition and Compositing", ICIP (2) 1997, pp. 366-369
