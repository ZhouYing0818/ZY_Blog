---
layout: post
title: "GATK合并的vcf转成plink输入数据、plink所有数据格式及转换方法"
description: "用昨天生成vcf文件转换plink输入文件，熟悉plink数据格式"
categories: [GWAS]
tags: [GATK plink] 
redirect_from:
  - /2020/09/17/
---
* Kramdown table of contents
{:toc .toc}

# 前言

  目前手上的数据是case-control和metadata分开的g.vcf数据，先挑了两个数据将流程跑顺，上文说了合并g.vcf为vcf的问题，接着进入GWAS分析需要将genotype和phenotype数据输入plink，plink的数据格式理解非常的重要，之前其实整理过一遍，也通过*2017-A tutorial on conducting genome-wide association studies   Quality control and statistical analysis-MPR-27-na*学习了一遍plink的基础使用，然而并不记得...

# 过程

## vcf转换plink的三种方式[^1]

* gatk3
    
  在gatk3中，提供了一个名为VariantsToBinaryPed的功能，可以将VCF格式转换为plink对应的二进制bed文件

* vcftools
   
  vcftools是操作vcf文件的一款常用工具，支持将vcf文件转换成plink对应的ped/map格式。<font color='red'>转出来.map格式跟plink不一样</font>

* plink
   
  plink1.9版本支持直接读取vcf/gen等多种文件格式，所以使用该版本时其实不需要专门进行格式转换，软件默认会将不同的格式转换为二进制bed文件格式。

~~~ bash
plink \
	--vcf input.vcf \
	--out output
~~~

* 输出文件为：output.bed, output.bim, output.fam
* <kbd>--recode</kbd>参数将输出结果调整为ped格式，可以更好的查看其转换的规则。加上<kbd>--record</kbd>之后的输出文件为：ped和map
* plink默认用下划线对样本名进行分隔，分隔的两个字段分别作为ped文件中的family id和sample id, 如果vcf中的样本名含有多个下划线，无法正确进行划分，软件会报错，此时可以修改<kbd>--id-delim</kbd>参数，该参数设定了分隔符，默认是下划线，可以设置成其他字符，以达到正确区分的目的。<font color='red'> vcf样本无分隔符，默认生成的family_id和sample_id是相同的</font>
* 通过参数指定family_id的设定方式，有两种参数，第一种如上述示例中的<kbd>--double_id</kbd>, 将family id和sample id保持相同

<img src='https://thumbnail0.baidupcs.com/thumbnail/7e5799568mcdf4ea251c22a929776d85?fid=1261248229-250528-884459955330906&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-pFJ4zfPv7TePUBwdGn0BxB6eGng%3d&expires=8h&chkbd=0&chkv=0&dp-logid=6045870458907205080&dp-callid=0&time=1600329600&size=c10000_u10000&quality=90&vuk=1261248229&ft=image' alt="" />

* plink另一种用法

~~~ bash
plink \
	--vcf input.vcf \
	--out output \
	--const-fid family_id
~~~

* 通过<kbd>--const-fid</kbd>将family id设置成一个常量，默认值是0

<img src='https://thumbnail0.baidupcs.com/thumbnail/42d308794ue44fe683b6ff6aa0c53c15?fid=1261248229-250528-1023211513453924&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-D3YakzgdSW7i4WQV%2bezgSZkih2E%3d&expires=8h&chkbd=0&chkv=0&dp-logid=6045870458907205080&dp-callid=0&time=1600329600&size=c10000_u10000&quality=90&vuk=1261248229&ft=image' alt="AltText" />

## plink的两种输入格式

### ped&map
* ped
    > FID
    > IID
    > 父亲ID
    > 母亲ID
    > 性别：1--男；2--女
    > 疾病状态：0 --无；1--不患病；2--患病
    > 第7列之后均为样本基因型信息

* map
    > 染色体号
    > SNPID
    > 遗传距离：0 --无；如果需要这个信息，可以根据物种的遗传图谱+插值法进行计算，或者根据1Mb=x cM的换算公式进行换算。
    > 物理距离

### bed&bim&fam
* bed：二进制的基因型数据

* bim类似map
    > 染色体号
    > SNPID
    > 遗传距离
    > 物理距离
    > 碱基1
    > 碱基2

* fam：等同ped前6列，记录表型数据
    > FID
    > IID
    > 父亲ID
    > 母亲ID
    > 性别: 1--男；2--女;0--性别未知
    > 表型: 1--对照；2--病例；'-9｜0'--表型缺失

    <font color='red'> **(plink仅能分析二等位位点，多等位位点会报错)**</font>

[^1]: <https://cloud.tencent.com/developer/article/1556166>
