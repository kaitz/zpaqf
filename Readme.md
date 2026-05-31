# zpaqf
Kaido Orav    
zpaqf v7.15.8f, 28.05.2026    

zpaqf is a fork of zpaq v7.15 (https://www.mattmahoney.net/dc/zpaq.html).    
All archives created by zpaqf can be unpacked by zpaq v7.15 and other compatible programs.    

### Differences between the zpaq (v7.15) and zpaqf (v7.15.8f) compression algorithm models.

The goal of the improvements is to add new filters, improve data compression, and, if possible, reduce the time required for compression.     
Changes in models are from level 3 (-m3) onwards. Some model parameters have been extended.

zpaqf detects 1, 4, 8, 24, 32 bit bmp images and 1 bit pbm, 8 bit pgm, 24 ppm images. 
Following types are used for special image models:    
IM1_PBM, IM8_PGM, IM24_PPM, IM1_BMP, IM4_BMP, IM8_BMP, IM24_BMP, IM32_BMP, IM_JPG, IM_AVI

If the file extension matches .bmp, .pbm, .pgm, or .ppm, then the image is checked for a valid header.
Images with byte width lower than 1024 are in solid block, assuming that they have the same width, otherwise each image is in individual block.
In method/level 5 images with bit depth 8 or 24 have special models that parse the file header to find the width. Otherwise, the byte width of the image is transmitted along with the block information.    
Method 5 also has a special model for text.    

On Windows Universal naming convention (UNC) paths are used by default when accessing files (a=add, x=extract). To extract with older (v7.15/v7.15.4f) versions of zpaq(f) ```-to``` command line option needs to be used. By renaming long paths to shorter version files can be extracted, but not into the original path. If command line option ```-to``` is not used then program prints an error message: ```path not found```    
For zpaqfranz, when extracting files with long path, use command: ```e myarchive.zpaq -longpath```   

Uses sparse file mode when unpacking files larger than 32 MB. This option is hardcoded and cannot be changed from the command line.    
### Transforms
There are two new post processors:
* WBPE - based on wbpe.cpp v1.1 - Preprocessor for text compression. (C) 2011, Dell Inc. Written by Matt Mahoney. GPL-3.
* LZMA - LZMA SDK 26.01 (2026-04-27) - public domain. (C) 2026 Igor Pavlov.

WBPE is used for text or as a prefilter for LZMA, and is used on level 3/4 (-m3,-m4). Can be selected from command line with ```-method xN,13```, where N is block size. CM model can fallow.

LZMA is used on level 3 (-m3). Can be selected from command line with ```-method xN,14,L,LC,LP,PB,FB,F,G```, where N is block size, L is LZMA compression level. No CM model.    
F and G are for filters. F=1 - for E8E9 filter, F=2 - for WBPE and G=1 for capital transform.    

### Mixer
There are two new parameters. Previously, the mixer default (```mN0,N1```) values were ```m8,24``` now (```mN0,N1,N2,N3```) ```m8,24,0,0```.    
New parameters selects how many upper bits (N2) of last byte are used as a context. Assuming that N0>8.    
If N3=1 then last byte is subtracted by 1.    

### SSE - Secondary Symbol Estimator
Has a new parameter. Before default (```sN0,N1,N2```) values for SSE were ```s8,32,255``` now  (```sN0,N1,N2,N3```) ```s8,32,255,0```.    
Last parameter (N3) selects how many last bytes to skip before we use them as contexts.   

### Indirect o1/o2 with or without n'th byte
Default values (```nN0,N1,N2,N3,N4```) for n contexts are ```n0,0,0,1,0```.    
Context predictors can be ICM or ISSE.    
Parameters from left to right are:    

    N0 0=ICM, 1=ISSE
    N1 indirect maskbits (0=32 reverse)
    N2 indirect stream shift bits (32=no indirect stream)
    N3 last N3'th byte (0=no bytes)
    N4 indirect width in bytes or order 1, 2 (0=1,1=2 bytes)

[The compression algorithm is described here](doc/The-ZPAQ-Compression-Algorithm.md) 

## Differences in methods
### Method 1

No change.

### Method 2

No change.

### Method 3

Method 3 uses special types to select the model of the identified data, such as:    

IM1_PBM, IM1_BMP: ```-method x0,c0.0.7.K.255``` (where J=byte width, K=J-1+999)    
IM4_BMP: ```-method x0,0c0.0.15.K.255``` (where J=byte width, K=J-1+999)    
IM8_PGM, IM8_BMP, IM24_PPM, IM24_BMP: ```-method x0,c0.0.255.K.255n1,8,0,0,1n1,8,0,3,1Mm``` (where J=byte width, K=J-1+999, if multiple files in blok M=a192)    
IM_JPG, IM_AVI: ```-method x0,c0.0.15.255i2n1,1,0,1,0```     

0..5: ```-method x0,0``` (no compression).    
6..199: and E8E9 ```-method x6,14,7,8,0,1,144,1``` (E8E9+LZMA)    
100..255: and text ```-method x6,14,7,8,0,P,144,2,1``` (where P=period?1:0) (WBPE(+CAP)+LZMA)    
200..255: and not text: ```-method x6,3ciJs8,32,85``` (where M=period<11?(period-1):0, J=1+M) (BWT with period if any).    
6..125: odd period and not text: ```-method x6,14,7,4,0,2,128``` (LZMA)    
6..125: period and not text: ```-method x6,14,7,LC,LP,PB,128``` (where G=log2(period-1)+/2, LC=8/(G>2?2:G), LP=G/(period>=256?2:1), PB=G-(period>12?1:0)) (LZMA)    
110..199: ```-method x6,14,7,8,0,0,128``` (LZMA)    
6..199: ```-method x6,14,7,4,0,2,128``` (LZMA)    
```
Different unique values for period data
Period LC, LP, PB
   3:   4,  0,  2
   4:   8,  1,  1
  10:   4,  2,  2
  14:   4,  2,  1
  34:   4,  3,  2
 130:   4,  4,  3
 256:   4,  2,  3
```

### Method 4

Method 4 uses special types to select the model of the identified data, such as:    

IM1_PBM, IM1_BMP: ```-method x0,c0.0.7.K.255m1``` (where J=byte width, K=J-1+999)     
IM4_BMP: ```-method x0,c0.0.15.K.255i2n0,4,0,1,0m16,10``` (where J=byte width, K=J-1+999)     
IM8_PGM, IM8_BMP: ```-method x0,c0.0.255.K.255c0.0.255.L.255t8s16,20,255,M``` (where J=byte width, K=J-1+999, L=J*2-1+999, M=J-2)     
IM24_PPM, IM24_BMP: ```-method x0,c0.3.255i2c0.0.511.K.255n1,8,0,3,1Mm11,24,3``` (where J=byte width, K=J-1+999, if multiple files in blok M=a192)     
IM32_BMP: ```-method x0,c0.4.255i2,3,3c0.0.511.K.255m11,24,3s16,24,255,3``` (where J=byte width, K=J-1+999)     
IM_JPG: ```-method x0,c0.0.15.255i2,1n1,1,0,1,0```     
IM_AVI: ```-method x0,c0.0.15.255i2,1n1,1,0,1,0```     

0..4: ```-method x0,0``` (no compression).    
5..5: ```-method x6,1.4.0.3.25``` (fast LZ77)    
6..11: ```-method x6,2.5.0.7.27.1c0.0.511``` (order 1 LZ77 with SA, minimum match length 5).    
12..24: ```-method x6,0ci1,1,1,1,2awm12,24,4```    
12..224 and E8E9: ```-method x6,4ci1,1,1,3an0,18,1,1,1m12,30,4```    
25..224: ```-method x6,0ci1,1,1,3an0,17,2,1,1m12,24,4```    
25..224: and text: ```-method x6,0c256.0.255i2,1,1,2aw2,65,26,223,191,0m11,10,4,1```    
25..224: and text with indirect: ```-method x6,0c256.0.255i2,1,3aw1,65,26,223,191,0n1,24,8,1,1m11,10,4,1```    
120..225: text or text like: ```-method x6,13ci2,3,2n1,24,8,3,1a24,1,1ts16,20,255``` (WBPE+CM)    
225..255: ```-method x6,3ciN``` (where M=period<10?period:0, N=1+M) (BWT+CM).    

### Method 5

Method 5 uses special types to select the model of the identified data, such as:

IM1_PBM, IM1_BMP: ```-method x0,c0.0.7.K.255i2c0.0.15.L.255i2m10,4,0``` (where J=byte width, K=J-1+999, L=J\*2-1+999)     
IM4_BMP: ```-method x0,c0.0.15.K.255i2c0.0.15.L.255i2n0,4,0,1,0m10,4,0s16,24,255,M``` (where J=byte width, K=J-1+999, L=J*2-1+999, M=J-2)     
IM8_PGM: ```-method x0,12```     
IM8_BMP: ```-method x0,11```     
IM24_PPM: ```-method x0,9```     
IM24_BMP: ```-method x0,8```     
IM32_BMP: ```-method x0,c0.4.255i2,3,3c0.0.511.K.255m11,24,3s16,24,255,3``` (where J=byte width, K=J-1+999)     
IM_JPG: ```-method x0,c0.0.7.255i2,1s16,18,63```     
IM_AVI: ```-method x0,c0.0.31.511i2,1ams16,18,63```     

0..2: ```-method x0,0``` (no compression)    
3..255 and text with brackets: ```-method x0,10,1```    
3..255 and text: ```-method x0,10,0```    
3..255 and E8E9: ```-method x3,4c256ci1,1,1,1,1,1,2ac0,2,0,255i1c0,3,0,0,255i1c0,4,0,0,0,255i1n1,18,1,1,1mm16ts19,15,255,2t0```    
3..255 no periodic models: ```-method x2,0w1i1c256ci1,1,1,1,1,1,2ac0,2,0,255i1c0,3,0,0,255i1c0,4,0,0,0,255i1mm16ts19t0```    
3..255 high period (>30): ```-method x0,0w1i1c256ci1,1,1,1,1,1,2ac0,0,K,255i1c0,Ji1c0,0,L,255i1mm16ts19t0``` (where J=period, K=J+999, L=J*2+999, ...)    
3..255 mid period (11-29): ```-method x3,0w1i1c256ciK,Jac0,0,L,255i1c0,Ji1mm16ts19t0``` (where J=period, K=J/2, L=J+999)    
3..255 low period, high period(1-10,>30): ```-method x6,0w1i1c256ciJ,Kn,Lac0,0,Y,255i1c0,Wi1c0,0,M,255i1c0,Li1mm16ts19t0``` (where L=low period J=2, Kn=J+1, Kn=K(n-1)+1 ... while Kn<L-K(n-1), M=L+999; W=high period Y=W+999)    

## Larger internal models
[Large models for different data types](doc/Large-models.md)

## Large tests
[Large test on 6 corpuses](doc/Tests.md)
#### BOOK1
<pre>
Method     v7.15.8f     v7.15
---------  --------   --------
-method 1   344197     344197
-method 2   315778     315778
-method 3   251793     215479
-method 4   205797     210188
-method 5   199094     200369
</pre>
