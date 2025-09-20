# zpaqf vs zpaq
Kaido Orav    
zpaqf v7.15.5f, 20.09.2025    

### Differences between the zpaq (v7.15) and zpaqf (v7.15.5f) compression algorithm models.

zpaqf detects 1, 4, 8, 24, 32 bit bmp images and 1 bit pbm, 8 bit pgm, 24 ppm images. 
Following types are used for special image models:
IM1_PBM, IM8_PGM, IM24_PPM, IM1_BMP, IM4_BMP, IM8_BMP, IM24_BMP, IM32_BMP, IM_JPG

If file extension matches to .bmp, .pbm, .pgm, .ppm then image header is tested for valid header.
Images with byte width lower than 1024 are in solid block assuming that they have the same width, otherwise each image is in individual block.
In method 5 images with bit depth 8, 24 have special models that parse the file header to find the width. Otherwise, the byte width of the image is transmitted along with the block information.    
Method 5 also has a special model for text.    

On Windows UNC paths are used by default when accessing files (a=add, x=extract). To extract with older versions of zpaq ```-to``` command line option needs to be used. By renaming long paths to shorter version files can be extracted, but not into the original path. If command line option ```-to``` is not used then program prints an error message: ```path not found```    
For zpaqfranz when extracting files with long path use command: ```e myarchive.zpaq -longpath```    

### Mixer
Has a two new parameters. Before default values for mixer was ```m8,24``` now ```m8,24,0,0```.    
Last two parameters (N2, N3) selects how many upper bits (N3) of last byte are used as a context. Assuming that N0>8. If N3=1 then last byte is subtracted by 1.    

### SSE - Secondary Symbol Estimator
Has a new parameter. Before default values for SSE was ```s8,32,255``` now ```s8,32,255,0```.    
Last parameters selects how many last bytes to skip before we use them as contexts.   

### Indirect o1/o2 with or without n'th byte
Default values for n contexts are ```n0,0,0,1,0```.    
Context predictors can be ICM or ISSE.    
Parameters from left to right are:    

    N0 0=ICM, 1=ISSE
    N1 indirect maskbits (0=32 reverse)
    N2 indirect stream shift bits (32=no indirect stream)
    N3 last N3'th byte (0=no bytes)
    N4 indirect width in bytes or order 1, 2 (0=1,1=2 bytes)

### Differences in method 1

No change.

### Differences in method 2

No change.

### Differences in method 3

Method 3 uses special types to select the model of the identified data, such as:    

IM1_PBM, IM1_BMP: ```-method x0,c0.0.7.K.255``` (where J=byte width, K=J-1+999)    
IM4_BMP: ```-method x0,0c0.0.15.K.255``` (where J=byte width, K=J-1+999)    
IM8_PGM, IM8_BMP, IM24_PPM, IM24_BMP: ```-method x0,c0.0.255.K.255n1,8,0,0,1n1,8,0,3,1Mm``` (where J=byte width, K=J-1+999, if multiple files in blok M=a192)    
IM32_BMP: ```-method x0,c0.4.255i2,3,3c0.0.511.K.255m11,24,3s16,24,255,3``` (where J=byte width, K=J-1+999)     
IM_JPG: ```-method x0,c0.0.7.255i2,1m```     

0..5: ```-method x0,0``` (no compression).    
6..11: ```-method x6,1.4.0.3.25``` (fast LZ77).    
12..159: and not text: ```-method x6,2.12.0.7.27.1c0.0.511i2s8,32,65``` (order 2 LZ77 with SA).    
160..255: or text and 12..255: ```-method x6,3ciJs8,32,85``` (where M=period<10?period/2:0, J=1+M) (BWT with period if any).    

### Differences in method 4

Method 4 uses special types to select the model of the identified data, such as:    

IM1_PBM, IM1_BMP: ```-method x0,c0.0.7.K.255m1``` (where J=byte width, K=J-1+999)     
IM4_BMP: ```-method x0,c0.0.15.K.255i2n0,4,0,1,0m16,10``` (where J=byte width, K=J-1+999)     
IM8_PGM, IM8_BMP: ```-method x0,c0.0.255.K.255c0.0.255.L.255t8s16,20,255,M``` (where J=byte width, K=J-1+999, L=J*2-1+999, M=J-2)     
IM24_PPM, IM24_BMP: ```-method x0,c0.3.255i2c0.0.511.K.255n1,8,0,3,1Mm11,24,3``` (where J=byte width, K=J-1+999, if multiple files in blok M=a192)     
IM32_BMP: ```-method x0,c0.4.255i2,3,3c0.0.511.K.255m11,24,3s16,24,255,3``` (where J=byte width, K=J-1+999)     
IM_JPG: ```-method x0,c0.0.7.255i2,1s16,18,63```     

0..4: ```-method x0,0``` (no compression).    
5..5: ```-method x6,1.4.0.3.25``` (fast LZ77)    
6..11: ```-method x6,2.5.0.7.27.1c0.0.511``` (order 1 LZ77 with SA, minimum match length 5).    
12..24: ```-method x6,0ci1,1,1,1,2awm12,24,4```    
12..224 and E8E9: ```-method x6,4ci1,1,1,3an0,18,1,1,1m12,30,4```    
25..224: ```-method x6,0ci1,1,1,3an0,17,2,1,1m12,24,4```    
25..224: and text: ```-method x6,0c256.0.255i2,1,1,2aw2,65,26,223,191,0m11,10,4,1```    
25..224: and text with indirect: ```-method x6,0c256.0.255i2,1,3aw1,65,26,223,191,0n1,24,8,1,1m11,10,4,1```    
225..255: ```-method x6,3ciN``` (where M=period<10?period:0, N=1+M) (BWT).    

### Differences in method 5

Method 5 uses special types to select the model of the identified data, such as:

IM1_PBM, IM1_BMP: ```-method x0,c0.0.7.K.255i2c0.0.15.L.255i2m10,4,0``` (where J=byte width, K=J-1+999, L=J\*2-1+999)     
IM4_BMP: ```-method x0,c0.0.15.K.255i2c0.0.15.L.255i2n0,4,0,1,0m10,4,0s16,24,255,M``` (where J=byte width, K=J-1+999, L=J*2-1+999, M=J-2)     
IM8_PGM: ```-method x0,12```     
IM8_BMP: ```-method x0,11```     
IM24_PPM: ```-method x0,9```     
IM24_BMP: ```-method x0,8```     
IM32_BMP: ```-method x0,c0.4.255i2,3,3c0.0.511.K.255m11,24,3s16,24,255,3``` (where J=byte width, K=J-1+999)     
IM_JPG: ```-method x0,c0.0.7.255i2,1s16,18,63```     

0..2: ```-method x0,0``` (no compression)    
3..255 and text with brackets: ```-method x0,10,1```    
3..255 and text: ```-method x0,10,0```    
3..255 and E8E9: ```-method x3,4c256ci1,1,1,1,1,1,2ac0,2,0,255i1c0,3,0,0,255i1c0,4,0,0,0,255i1n1,18,1,1,1mm16ts19,15,255,2t0```    
3..255 no periodic models: ```-method x2,0w1i1c256ci1,1,1,1,1,1,2ac0,2,0,255i1c0,3,0,0,255i1c0,4,0,0,0,255i1mm16ts19t0```    
3..255 high period (>30): ```-method x0,0w1i1c256ci1,1,1,1,1,1,2ac0,0,K,255i1c0,Ji1c0,0,L,255i1mm16ts19t0``` (where J=period, K=J+999, L=J*2+999, ...)    
3..255 mid period (11-29): ```-method x3,0w1i1c256ciK,Jac0,0,L,255i1c0,Ji1mm16ts19t0``` (where J=period, K=J/2, L=J+999)    
3..255 low period, high period(1-10,>30): ```-method x6,0w1i1c256ciJ,Kn,Lac0,0,Y,255i1c0,Wi1c0,0,M,255i1c0,Li1mm16ts19t0``` (where L=low period J=2, Kn=J+1, Kn=K(n-1)+1 ... while Kn<L-K(n-1), M=L+999; W=high period Y=W+999)    

### Large models for different data types.

**-method x0,8 -method x0,9**    
24 bit image model
```
(24 bit bmp, ppm model)
comp 17 17 0 3 20 (hh hm ph pm n)
    0 const 160
    1 cm $1+16 255
    2 cm $1+16 255
    3 cm $1+16 255
    4 cm $1+16 255
    5 cm $1+16 255
    6 icm $1+16
    7 icm $1+16
    8 icm $1+16
    9 cm $1+20 255
    10 cm $1+20 255
    11 cm 16 255
    12 match 21 23
    13 mix 16  0 13 16 255
    14 mix 11  0 14 20 255
    15 mix2 0 13 14 40 0
    16 mix  0  0 15 32 0
    17 mix2 0 15 16 40 0
    18 sse 16 17  8 255
    19 mix2 8 17 18 40 255
hcomp
    *c=a (save in rotating buffer)
    a=c a== 0 if (compute lookup ilog table)
      d= 255 *d=0 d++ *d=0
      a= 1 a<<= 16 r=a 0
      a= 7 a*= 100 a+= 74 a*= 100 a+= 54 a*= 100 a+= 10 a*= 100 a+= 2 r=a 1
      a= 14 a*= 100 a+= 15 a*= 100 a+= 57 a*= 100 a+= 76 c=a
      d= 2
      do
        a=d a*= 2 a-- b=a
        a=r 1 a/=b b=a
        a=c a+=b c=a
        a=d a+= 255 d=a a=c a>>= 24 *d=a a=d a-= 255 d=a
        d++
      a=r 0 b=a a=d a<b while
```
bmp
```      
      c=0 a= 255 r=a 0  endif
```
ppm
```
      c=0 a= 255 r=a 0 a= 2 r=a 20 endif
```
bmp
```
    a=c a== 3 if
      b=0 a=*b
      a== 0 ifnot b++ b++ a=*b a>>= 8 b-- a+=*b a++ a++ endif
      a+= 20
      r=a 19
    endif
    a=r 19 a==c if b=c a=*b a<<= 8 b-- a+=*b a*= 3 a+= 3 a|= 3 a^= 3 r=a 0 endif (r0=w)
```
ppm
```
    a=r 19 a== 0 if
      a=c a== 4 if
        a= 3 r=a 20 (header)
      endif
      a=*c a== 35 if (#)
        a= 2 r=a 20 (read comment)
      endif
      a=r 20 a== 6 if
        a=r 21 a*= 3  r=a 0 (header done, r0=w, set width)
        r=a 19              (end)
      else
        a> 2 if  (not comment)
          a== 3 if (width)
              a=*c a== 32 if
                  a= 4 r=a 20
              else
                  a=r 21 a*= 10 b=a a=*c a-= 48 a+=b r=a 21 (r21=r21*10+c-'0')
              endif
          else
              a== 4 if (height)
                  a=*c a== 10 if (line ended)
                      a= 5 r=a 20
                  endif
              else
                  a== 5 if    (limit)
                      a=*c a== 10 if (line ended)
                          a= 6 r=a 20
                      endif
                  endif
              endif
          endif
        else
          a=*c a== 10 if
              a= 3 r=a 20 (line ended, read val)
          endif
        endif
      endif
    endif
```
```
    a=c a%= 3 r=a 1                              (r1=color)
    b=c b-- b-- a=*b r=a 2                       (r2=buf(3))
    a=c a++ b=r 0 a-=b b=a a=*b r=a 3            (r3=buf(w))
    a=c a-- a-- b=r 0 a-=b b=a a=*b r=a 4        (r4=buf(w+3))
    a=c a+= 4 b=r 0 a-=b b=a a=*b r=a 5          (r5=buf(w-3))
    a=r 2 b=r 3 a+=b b=r 4 a+=b b=r 5 a+=b r=a 6 (r6=mean)
    a=r 2 a*=a b=a a=r 3 a*=a a+=b b=a a=r 4 a*=a
    a+=b b=a a=r 5 a*=a a+=b b=a a=r 6 a*=a a/= 4
    b<>a a-=b a>>= 2 r=a 7                       (r7=var)
    a=r 6 a>>= 2 r=a 6                           (r6>>=2)
    a=r 7 a+= 255 d=a a=*d r=a 8                 (r8=logvar)
          
    b=c a=*b r=a 9 (r9=buf(1))
    b=c b-- a=*b r=a 10 (r10=buf(2))
    a=c a-= 5 b=a a=*b r=a 12 (r12=buf(6))

    a=c a+= 7 b=r 0 a-=b a-=b b=a a=*b r=a 14    (r14=buf(w*2-6))
    a=c a+= 4 b=r 0 a-=b a-=b b=a a=*b r=a 15    (r15=buf(w*2-3))
    a=c a-= 5 b=r 0 a-=b a-=b b=a a=*b r=a 16    (r16=buf(w*2+6))
    a=c b=r 0 a-=b b=a a=*b r=a 17               (r17=buf(w+1))
    a=c a+= 3 b=r 0 a-=b b=a a=*b r=a 18         (r18=buf(w-2))

    d=0 do *d=0 d++ a=d a< 12 while

    ( =(buf(3)+buf(w))>>3, buf(1)>>4, buf(2)>>4 ) 
    d=0 a=r 2 b=r 3 a+=b a>>= 3 hashd a=r 9 a>>= 4 hashd a=r 10 a>>= 4 hashd d++ 
    a=r 3 b=r 4 a-=b b=r 2 a+=b hashd d++
    a=r 2 a*= 2 b=r 12 a-=b hashd d++
    a=r 4 a*= 2 b=r 16 a-=b hashd d++
    a=r 9 b=r 18 a-=b b=r 5 a+=b hashd d++
    a=r 3 b=r 15 a-=b b=r 5 a+=b hashd d++
    a= 24 a*= 16 b=a a=r 8 a<<= 1 a&=b b=a a=r 6 a>>= 1 a|=b hashd d++

    b=r 17 a=r 9 a-=b b=r 3 a+=b hashd d++
    a=r 9 hashd a=r 10 hashd d++
    a=r 2 hashd a=r 9 hashd a=r 10 hashd d++
    a=r 3 hashd a=r 9 hashd a=r 10 hashd d++
    a=r 3 hashd a=r 9 hashd d++ d++

    d=0
    do
      a=r 1 hashd d++
    a=d a< 12 while

    d= 12 a=*d a*= 192 a+=*c a++ *d=a d++                             (match)
    a=r 9 a>>= 4 a*= 3 b=r 1 a+=b a<<= 9 *d=a d++                     (mix)
    a=r 1 a<<= 9 *d=a                                                 (mix)
    d++ d++ d++ d++
    a=r 3 (b=r 4 a-=b) b=r 2 a+=b a>>= 3 a*= 3 b=r 1 a+=b a<<= 9 *d=a (sse)
    c++
    halt
end
```
**-method x0,11 -method x0,12**    
8 bit image model
```
(8 bit bmp, pgm model)
comp 17 17 0 3 19 (hh hm ph pm n)
    0 const 160
    1 cm 20 255
    2 cm 20 255
    3 cm 20 255
    4 cm 20 255
    5 cm 20 255
    6 icm 16
    7 icm 16
    8 icm 16
    9 cm 20 255
    10 cm 20 255
    11 cm 11 255

    12 mix 16  0 12 16 255
    13 mix 11  0 13 20 255
    14 mix2 0 12 13 40 0
    15 mix  0  0 14 32 0
    16 mix2 0 14 15 40 0
    17 sse 16 16  8 255
    18 mix2 8 16 17 40 255
hcomp
    *c=a (save in rotating buffer)
```
pgm
```
    a=c a== 0 if
       a= 2 r=a 20
    endif
```
bmp
```
    a=c a== 3 if
        b=0 a=*b
        a== 0 ifnot b++ b++ a=*b a>>= 8 b-- a+=*b a++ a++ endif
        a+= 20
        r=a 19
    endif
    a=r 19 a==c if b=c a=*b a<<= 8 b-- a+=*b r=a 0 endif (r0=w)
```
pgm
```
    a=r 19 a== 0 if
        a=c a== 4 if
            a= 3 r=a 20 (header)
        endif
        a=*c a== 35 if (#)
            a= 2 r=a 20 (read comment)
        endif
        a=r 20 a== 6 if
            a=r 21 (a*= 3)  r=a 0 (header done, set width)
            r=a 19              (end)
        else
            a> 2 if  (not comment)
                a== 3 if (width)
                    a=*c a== 32 if
                        a= 4 r=a 20
                    else
                        a=r 21 a*= 10 b=a a=*c a-= 48 a+=b r=a 21 (r21=r21*10+c-'0')
                    endif
                else
                    a== 4 if (height)
                        a=*c a== 10 if (line ended)
                            a= 5 r=a 20
                        endif
                    else
                        a== 5 if    (limit)
                            a=*c a== 10 if (line ended)
                                a= 6 r=a 20
                            endif
                        endif
                    endif
                endif
            else
                a=*c a== 10 if
                    a= 3 r=a 20 (line ended, read val)
                endif
            endif
        endif
    endif
```
contexts
```
    b=c a=*b r=a 2 (r2=buf(1))
    a=c a++ b=r 0 a-=b b=a a=*b r=a 3 (r3=buf(w))
    a=c b=r 0 a-=b b=a a=*b r=a 4 (r4=buf(w+1))
    a=c a+= 1 b=r 0 a-=b b=a a=*b r=a 5 (r5=buf(w-1))
    a=r 2 b=r 3 a+=b b=r 4 a+=b b=r 5 a+=b r=a 6 (r6=mean)
    a=r 2 a*=a b=a a=r 3 a*=a a+=b b=a a=r 4 a*=a a+=b b=a a=r 5 a*=a a+=b b=a
    a=r 6 a*=a a/= 4 b<>a a-=b a>>= 2 r=a 7 (r7=var)
    a=r 6 a>>= 2 r=a 6 (r6>>=2)
    a=c a++ b=r 0 a-=b a-=b b=a a=*b r=a 8 (r8=buf(w*2))

    b=c a=*b r=a 9 (r9=buf(1))
    b=c b-- a=*b r=a 10 (r10=buf(2))
    a=c a-= 3 b=a a=*b r=a 12 (r12=buf(4))

    a=c a+= 2 b=r 0 a-=b a-=b b=a a=*b r=a 14 (r14=buf(w*2-2))
    a=c a+= 1 b=r 0 a-=b a-=b b=a a=*b r=a 15 (r15=buf(w*2-1))
    a=c a-= 1 b=r 0 a-=b a-=b b=a a=*b r=a 16 (r16=buf(w*2+2))
    a=c b=r 0 a-=b b=a a=*b r=a 17 (r17=buf(w+1))
    a=c a+= 3 b=r 0 a-=b b=a a=*b r=a 18 (r18=buf(w-2))

    d=0 do *d=0 d++ a=d a< 12 while
    ( =(buf(1)+buf(w))>>3, buf(1)>>4, buf(2)>>4 ) 
    d=0 a=r 2 b=r 3 a+=b a>>= 3 hashd a=r 9 a>>= 4 hashd a=r 10 a>>= 4 hashd d++ 
    a=r 3 b=r 4 a-=b b=r 2 a+=b hashd d++
    a=r 2 a*= 2 b=r 12 a-=b hashd d++
    a=r 4 a*= 2 b=r 16 a-=b hashd d++
    a=r 9 b=r 18 a-=b b=r 5 a+=b hashd a=r 8 hashd d++
    a=r 3 b=r 15 a-=b b=r 5 a+=b hashd a=r 8 hashd d++
    a=r 6 a>>= 1 hashd a=r 8 hashd d++
    b=r 17 a=r 9 a-=b b=r 3 a+=b hashd d++
    a=r 9 hashd a=r 10 hashd d++
    a=r 2 hashd a=r 9 hashd a=r 10 hashd d++
    a=r 3 hashd a=r 9 hashd a=r 10 hashd d++
    a=r 3 hashd a=r 9 hashd d++

    d= 12
    a=r 9 b=r 2 a-=b b=r 3 a+=b a<<= 9 *d=a d++ (mix W-WW+N)
    a=r 2 a>>= 6 a<<= 9 *d=a (mix, high 2 bits of buf(1))

    d++ d++ d++ d++

    a=r 3 b=r 2 a+=b a<<= 9 *d=a (sse)
    c++
    halt
end
```
**-method x0,10,0 -method x0,10,1**    
Text model    
Last parameter in method selects bracket (1).

no brackets
```
comp 10 9 0 0 22
```
brackets
```
comp 10 9 0 0 23
```
```
    0 const 158
    1 icm 5
    2 isse 13 1
    3 isse $1+15 2
    4 isse $1+17 3
    5 isse $1+19 4
    6 isse $1+19 5
    7 match $1+22 $1+23
    8 isse $1+20 6
    9 icm $1+17
    10 isse $1+19 9
    11 icm $1+15
    12 icm $1+15
    13 icm $1+14
```
brackets
```
    14 icm $1+14
    15 icm $1+15
    16 icm $1+15
    17 icm $1+16
    18 mix 17 0 18 24 255
    19 mix 15 0 19 10 255
    20 mix2 0 18 19 24 0
    21 sse 18 20 32 255
    22 mix2 0 20 21 16 0
```
no brackets
```
    14 icm $1+15
    15 icm $1+15
    16 icm $1+16
    17 mix 17 0 17 24 255
    18 mix 15 0 18 10 255
    19 mix2 0 17 18 24 0
    20 sse 18 19 32 255
    21 mix2 0 19 20 16 0
```
```
hcomp
    b=a a=c a== 0 ifl
      (fill 2 bit char map[256])
      d= 255 d++
      a=0 do *d= 3 d++ a++ a< 48 while
      a=0 do *d= 2 d++ a++ a< 48 while
      a=0 do *d= 1 d++ a++ a< 112 while
      a= 255 a+= 49 d=a
      a= 49 do *d= 1 d++ a++ a< 59 while
      b= 255 b++
      a= 0 a+=b d=a *d= 2 a= 2 a+=b d=a *d= 1 a= 5 a+=b d=a *d= 0 a= 6 a+=b d=a *d= 1
      a= 7 a+=b d=a *d= 2 a= 10 a+=b d=a *d= 0 a= 11 a+=b d=a *d= 0 a= 12 a+=b d=a *d= 1
      b= 0
      a= 0 a+= 255 a++ a+= 255 a++ r=a 10 (r10=ind[0])
    endif
    a=b
    c++ *c=a a= 1
    a+= 255
    b=a a=*c a+=b a++ d=a b=*d
    a=r 1 a<<= 2 a+=b r=a 1      (b2stream=r1=(r1<<2)+map[byte] stream of 2 bit chars)
    a=r 2 a<<= 1 r=a 2           (words<<1)
    b=c a=0
    b-- a=*b b++ a== 10 if a=*b r=a 8  endif a=0 (r2=firstchar)
    b=c b-- a=*b r=a 11(byte2)  a=*c r=a 12(byte)                      (indirect byte)
    a=r 10 b=a a=r 11 a+=b (offs) d=a a=*d a<<= 8 b=a a=r 12 a|=b *d=a (ind[byte2]=(ind[byte2]<<8)|byte)
    a=r 10 b=a a=r 12 a+=b (offs) d=a a=*d r=a 13                      (r13=ind[byte])
    b=c a=0
    d= 2 hash *d=a b--             (1)
    d++ hash *d=a b--              (2)
    d++ hash *d=a b--              (3)
    d++ hash *d=a b--              (4)
    d++ hash b--                   (5)
        hash *d=a b--              (6)
    d++ hash b--                   (7)
        hash b-- *d=a              (8 match)
    d++ hash b--                   (9)
        hash *d=a                  (10)
    d++ a=*c a&~ 32                (lowercase words or dictionary encoded)
    a> 64 if
      a< 91 if                     (a-z)
        d++ hashd d--              (word1)
        *d<>a a+=*d a*= 191 *d=a r=a 6  (r6=word0)
        a=r 2 a++ r=a 2            (words++)
      else
        a=*c a> 127 if             (a-z)
          d++ hashd d--            (word 1)
          *d<>a a+=*d a*= 191 *d=a r=a 6 (r6=word0)
          a=r 2 a++ r=a 2          (words++)
        else
          a=*d a== 0 ifnot         (word1=word0)
            d++ b=*d *d=a a=b d--
          endif
          *d=0
        endif
      endif
    else
      a=*d a== 0 ifnot
        d++ b=*d *d=a r=a 7 a=b r=a 5 d-- (r5=word2 r7=word1)
      endif
      *d=0
      b=c b-- a=*b a== 58 if
      a=r 4 a< 64 if a=r 3 a== 0 if
      a=*c a== 32 if             (:)
      a=r 2 a&= 4 a== 4 if
          a=r 7 r=a 14           (r14=cword)
        endif endif endif endif endif
      a=*c a== 46 if             (.)
         d++ a=*d a*= 53 *d=a d-- a=r 2 a<<= 1 r=a 2 a=r 1 a|= 254 r=a 1      ( shift word, words, b2stream )
         a=0 r=a 14
      endif
      a=*c a== 44 if             (,)
         a=0 a-- r=a 2 ( words=ffffffff)  a=r 1 a|= 240 r=a 1 (b2stream)
      endif
    endif
    d++
    ( hash(((b2stream&FFFFFFFC)<<6)+byte,words&15,firstchar) )
    d++ *d=0 a=0 a-= 4 b=a a=r 1 a&=b a<<= 6 b=*c a+=b
        hashd a=r 2 a&= 15 hashd  a=r 8 hashd
    ( hash(((b2stream&240)<<6)+byte,(ind[byte]>>8)&0xffff,words) )                     
    d++ *d=0 a=r 1 a&= 240 a<<= 4 b=*c a+=b hashd
        a= 1 a<<= 16 a-- b=a a=r 13 a>>= 8 a&=b hashd a=r 2 hashd
    ( hash(((b2stream&255)<<8)+byte,colonword?cword:word2) )    
    d++ *d=0 a=r 1 a&= 255 a<<= 8 b=*c a+=b *d=a hashd
        a=r 14 a== 0 if a=r 5 hashd else hashd endif                          
```
brackets
```
        ( hash(bracketcount,byte,(b2stream>>2)&255),firstchar )
        d++ a=*c a== 91 if a=r 3 a< 5 if a++ r=a 3 endif endif (brackets {[]})
        a== 123 if a=r 3 a< 5 if a++ r=a 3 endif endif
        a== 93 if a=r 3 a> 0 if a-- r=a 3 endif endif
        a== 125 if a=r 3 a> 0 if a-- r=a 3 endif endif
        b=c a=r 3 hash *d=a b-- a=*b hashd a=r 1 a>>= 2 a&= 255 hashd
        a=r 8 hashd                                                           
```
```
    ( hash(((words<<2)+(b2stream&3))*191+byte,word0) )
    d++ a=r 2 a<<= 2 b=r 1 b<>a a&= 3 a+=b a*= 191 b=*c a+=b *d=a a=r 6 hashd
    ( hash(buf(column+1),b2stream,firstchar, word1) )
    d++ a=*c a== 10 if (column)
        a=0 r=a 4 b=c b-- a=*b a== 10 if a=0 r=a 5 r=a 3 endif
      else
        a=r 4 a< 64 if a++ endif r=a 4
      endif
      b=c b<>a a-=b b<>a b++ a=r 1 hash *d=a a=r 8 hashd a=r 7 hashd
    ( hash(ind[byte]&0xffff,byte,b2stream&63) )
    d++ *d=0 a= 1 a<<= 16 a-- b=a a=r 13 a&=b hashd a=*c hashd
        a=r 1 a&= 63 hashd
    ( mix=b2stream )                                                   
    d++ a=r 1 a<<= 9 *d=a
    ( mix=(((b2stream&63)<<1)+(words&1))*byte )                                         
    d++ a=r 1 a&= 63 a<<= 1 b=r 2 b<>a a&= 1 a+=b b=a a=*c a*=b *d=a          
    d++
    ( sse=hash((b2stream<<4)+(words&15),word0) )
    d++ a=*c a<<= 14 b=c b-- b-- hash *d=a a=r 1 a<<= 4 b=r 2 b<>a
        a&= 15 a+=b hashd a=r 6 hashd                                         
    halt
end
```

## BOOK1
<pre>
Method     v7.15.v4     v7.15
---------  --------   --------
-method 1   344048     344197
-method 2   315629     315778
-method 3   212724     215479
-method 4   205648     210188
-method 5   199033     200369
</pre>
