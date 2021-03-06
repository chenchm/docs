docker镜像层之间的关系（UnionFS）

Docker镜像是由多个文件系统（只读层）叠加而成。当我们启动一个容器的时候，Docker会加载只读镜像层并在其上（译者注：镜像栈顶部）添加一个读写层。如果运行中的容器修改了现有的一个已经存在的文件，那该文件将会从读写层下面的只读层复制到读写层，该文件的只读版本仍然存在，只是已经被读写层中该文件的副本所隐藏。当删除Docker容器，并通过该镜像重新启动时，之前的更改将会丢失。在Docker中，只读层及在顶部的读写层的组合被称为[Union File System](https://docs.docker.com/terms/layer/#union-file-system)（联合文件系统）。

镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

对于文件的操作无非以下四种：

- 增加了某文件（文件夹也可以看做是文件，下面统写为文件）
- 删除了某文件
- 修改了某文件
- 某文件不变

在子镜像层中对文件进行以上操作时，对应变化为：

- 增加了某文件：则在子镜像层中直接增加该文件。比如父镜像层中没有/d.txt，而在子镜像有，那么子镜像中直接在根中加一个d.txt。
- 删除了某文件：在子镜像层中进行标注，比如删除了父镜像层中存在的/c.txt，那么在子镜像中，则生成一个/..c.txt的文件，表明这个文件已经被删除了。
- 修改了某文件：因为跟父镜像层的文件不同了，同增加了某文件（应该只是修改不同的部分？）。
- 某文件不变：父镜像层中有/a.txt，子镜像层中就不必再包含了。

docker镜像层是在本地分层存储的，根据具体驱动不同，最终实现的效果也不相同。

- vfs，vfs是采用全量的方式来存储镜像的。比如有a层，那么镜像A在vfs中存储有a层的所有文件。镜像B在vfs中将首先将a/b两层的文件进行拼合，得到拥有所有文件的完整镜像B，然后存在另一个目录中。以此类推。
- UnionFS，联合文件系统可以把不同目录下的文件融合到一个目录下。比如在目录dirA下有a.txt，dirB下有b.txt。联合文件系统可以对用户提供一个dir文件，里面包含了dirA和dirB下的内容，那么对于用户，就可以看到dir下有两个文件，a.txt和b.txt了。例如镜像A是根镜像，镜像B是子镜像，那么存储镜像A是需要存储完整的，但是在存储镜像B时，可以利用a层和b层之间的关系，只需要存储增量部分的内容即可。
