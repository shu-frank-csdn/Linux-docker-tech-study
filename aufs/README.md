README
===========================
该文件是用来描述关于Linux aufs 的原理
#AUFS例子
分层的AUFS的理解，可以类比Photoshop的图层，图层1,2,3,1上如果有一个红点，2上在不同的位置有个黑点，3上在与1上相同的位置有个绿点，那么结果就是只会看到绿点和黑点。

类比文件：
假设一个目录/XXX由自底向上的ABC三层组成,他们每一层都有一个和他们名字对应的文件(A=>A.txt, ..),同时他们都有一个叫common.txt文件,里面的内容分别是A,B,C,那么最终的结果是/XXX目录共有4个文件,A.txt, B.txt, C.txt和common.txt,而common.txt里的内容是C(最上层覆盖下层的)

例子:创建三个目录
```
mkdir -p /tmp/aufs/root
mkdir -p /tmp/aufs/layer1
mkdir -p /tmp/aufs/layer2
```
创建相应的文件：
```
echo 'root' > /tmp/aufs/root/root.txt
echo 'layer1' > /tmp/aufs/layer1/layer1.txt
echo 'layer2' > /tmp/aufs/layer2/layer2.txt
echo 'layer1, common' > /tmp/aufs/layer1/common.txt
echo 'layer2, common' > /tmp/aufs/layer2/common.txt
```
按照顺序mount起来，br后面的参数顺序越后面越底层：
```
mount -t aufs -o br=/tmp/aufs/layer2:/tmp/aufs/layer1:/tmp/aufs/root none /tmp/aufs/root
```

现在/tmp/aufs/root/下应该有4个文件,root.txt, layer1.txt, layer2.txt和common.txt,并且common.txt里面的内容应该是layer2.同时由于现在layer2是最上层,那么/tmp/aufs/root里面做修改会反映到/tmp/aufs/layer2,在/tmp/aufs/root里创建文件,/tmp/aufs/layer2里也会出现

那commit的时候，到底在做什么。再添加一层layer3，然后对/tmp/aufs/root做修改：
```
mkdir -p /tmp/aufs/layer3
umount /tmp/aufs/root
mount -t aufs -o br=/tmp/aufs/layer3:/tmp/aufs/layer2:/tmp/aufs/layer1:/tmp/aufs/root none /tmp/aufs/root
```
#Docker host file structure
镜像的存储结构主要分两部分，一是镜像ID之间的关联，一是镜像ID与镜像名称之间的关联，前者的结构体叫Graph，后者叫TagStore.
- /var/lib/graph/<image id> 下面没有layer目录，只有每个镜像的json描述文件和layersize大小
- /var/lib/docker/repositories-aufs TagStore的存储地方，里面有image id与reponame ，tag之间的映射关系. aufs是driver名
- /var/lib/docker/aufs/diff/<image id or container id> 每层layer与其父layer之间的文件差异，有的为空，有的有一些文件(镜像实际存储的地方)
- /var/lib/docker/aufs/layers/<image id or container id> 每层layer一个文件，记录其父layer一直到根layer之间的ID，每个ID一行。大部分文件的最后一行都一样，表示继承自同一个layer.
- /var/lib/docker/aufs/mnt/<image id or container id> 有容器运行时里面有数据(容器数据实际存储的地方,包含整个文件系统数据)，退出时里面为空
