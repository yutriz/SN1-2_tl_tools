## 准备
0. ROM
1. 工具
    * 解包/打包: [ndstool](https://github.com/devkitPro/ndstool)
    * 解压/压缩: [DSDecmp](https://gbatemp.net/download/dsdecmp.37134/)

## 步骤
0. Build
    ```
    git clone --recurse-submodules https://github.com/yutriz/SN1-2_tl_tools
    cd SN1-2_tl_tools/rextr
    mkdir build && cd build
    cmake ../ && make
    ```
1. 解包ROM
    ```
    ndstool -x rom_file -9 arm9.bin -7 arm7.bin -y9 y9.bin -y7 y7.bin -d data -y overlay -t banner.bin -h header.bin
    ```

2. 解压场景对话文件
    ```
    for scn in ./data/scnrts/*.rtz
    do
        DSDecmp $scn
        quickextr -i $scn
    done
    ```
    得到若干以.qe.json为后缀的文件
3. 翻译工作
4. 码点的重新映射
    ```
    nodupl -k json_key -c char_per_line *.qe.json > char_map.txt
    ```
    json_key: 翻译文本键名

    char_per_line: 每行字数（用于制作NFTR Font）

    char_map.txt无重复地储存了所有翻译用到的字符

    码点重新映射，假设游戏原本的码点范围为 row[0x10, 0x20] col[0x40, 0x50], 我们希望将用0x1040表示"中"，而"中"在UTF-8中以0xabcd表示，则需要在重新写入对话文件时进行 0xabcd -> 0x1040 的转换，将所有的转换（像0xabcd 0x1040）储存在new_conversion.txt中（NFTR的结构并不是特别复杂 http://problemkaputt.de/gbatek.htm#dscartridgenitrofontresourceformat, 但真改起来很繁琐）


4. 重新写入原文件并压缩
    ```
    for scn in ./data/scnrts/*.rtz
        quickrepack_enc -r $scn -i ${scn}.qe.json -c new_conversion.txt
        // quickrepack -r $scn -i ${scn}.qe.json (for english)
        dsdecmp -c lz10 $scn
    done
    ```
5. 打包ROM
    ```
    ndstool -c ROM_TLED -9 arm9.bin -7 arm7.bin -y9 y9.bin -y7 y7.bin -d data -y overlay -t banner.bin -h header.bin
    ```
6. DONE


## 效果
![sample_1](https://github.com/yutriz/SN1-2_tl_tools/blob/main/imgs/1.gif)
![sample_2](https://github.com/yutriz/SN1-2_tl_tools/blob/main/imgs/2.gif)

## TODO
1. NFTR生成程序 Font优化
2. 一键脚本
3. game system翻译