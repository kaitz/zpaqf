# Tests

## Test computer parameters		
CPU: 	Intel(R) Core(TM) i5-4460 CPU @ 3.20GHz	   
RAM: 	4x8GB PC3-12800 (800 MHz)	
		
Measured only compression time and ratio.		   
		
## Tested corpuses		
* Silesia	http://www.mattmahoney.net/dc/silesia.html
*	Calgary	https://www.mattmahoney.net/dc/calgary.html
*	Maximum Compression	https://web.archive.org/web/20210812200847/https://maximumcompression.com/data/files/index.html
*	Large Text Compression Benchmark	http://www.mattmahoney.net/dc/text.html
*	10 GB Compression Benchmark	http://www.mattmahoney.net/dc/10gb.html
*	Lossless Photo Compression Benchmark	http://qlic.altervista.org/LPCB-data.html

## Tables
URL: https://docs.google.com/spreadsheets/d/1F4JL0_l8xsE6CACATL2Hdd7Hl5-wMnAblIp6oPxcVjU/

### enwik
Compression (bytes)        

                zpaq v7.15   zpaqf v7.15.6  diff
    enwik6 -m3    261312      258812           2500
    enwik6 -m4    237901      235832           2069
    enwik6 -m5    229974      220107           9867
    enwik7 -m3    2375046     2345417         29629
    enwik7 -m4    2188214     2188241           -27
    enwik7 -m5    2091360     1996501        94859
    enwik8 -m3    21980368    21662343       318025
    enwik8 -m4    20740507    20626058       114449
    enwik8 -m5    19625017    18708777       916240
    enwik9 -m3    189875989   187194616     2681373
    enwik9 -m4    179455248   177830112     1625136
    enwik9 -m5    168590740   159171546     9419194
                
Time (sec)

                zpaq v7.15  zpaqf v7.15.6    diff
    enwik6 -m3    0,297      0,359           -0,062
    enwik6 -m4    0,889      0,92            -0,031
    enwik6 -m5    2,995      3,448           -0,453
    enwik7 -m3    3,011      3,682           -0,671
    enwik7 -m4    9,844      8,971            0,872
    enwik7 -m5    30,763     33,462          -2,699
    enwik8 -m3    22,136     26,551          -4,415
    enwik8 -m4    69,046     62,557           6,489
    enwik8 -m5    215,999    233,455        -17,456
    enwik9 -m3    90,746     108,608        -17,862
    enwik9 -m4    290,24     262,113         28,127
    enwik9 -m5    949,157    999,077        -49,92
