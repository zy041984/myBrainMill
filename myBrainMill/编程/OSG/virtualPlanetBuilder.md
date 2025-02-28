# 编译
0.9.8版本，使用osg3.4.0和gdal2.0.0用vs2022编译
# 使用
把高程tif和图片tif输出为带有纹理的ive文件
`.\osgdem.exe -d "E:\data\dem\30\srtm30plus_stripped.tif" -t "E:\data\blueMarble\June\world.topo.bathy.200406.3x21600x10800.png" --cs WGS84 -o "E:\data\srtm30_bMJune.ive" -l 4 8 --PagedLOD --no-mip-mapping -v 100`
失败

`.\osgdem.exe -d "E:\data\dem\30\srtm30plus_stripped.tif" -l 7 -v 100000 -o "E:\data\l7v100000\srtm30_bMJune.ive" `
可以输出一个白色图片，但是高程范围很小，-v好像没作用