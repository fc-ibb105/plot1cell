# plot1cell: a package for advanced single cell data visualization

This package allows users to visualize the single cell data on the R objects or output files generated by the popular tools such as Seurat. It is currently under active development. 

## Installation
plot1cell R package can be easily installed from Github using devtools. Please make sure you have installed Seurat 4.0, circlize and ComplexHeatmap packages.
```
devtools::install_github("TheHumphreysLab/plot1cell")
## or the development version, devtools::install_github("HaojiaWu/plot1cell")
```

## Usage
We provide some example codes to help generate figures from user's provided Seurat object. The Seurat object input to plot1cell should be a final object with completed clustering and cell type annotation. If a seurat object is not available, we suggest to use the demo data from Satija's lab (https://satijalab.org/seurat/articles/integration_introduction.html). To demonstrate the plotting functions in plot1cell, we re-created a Seurat object from our recent paper <a href="https://www.pnas.org/doi/10.1073/pnas.2005477117">Kirita et al, PNAS 2020</a> by integrating the count matrices we uploaded to GEO (GSE139107).
```
iri.integrated <- Install.example() 

# Please note that this Seurat object is just for demo purpose and 
# is not exactly the same as we published on PNAS.
# It take about 2 hours to run in a linux server with 500GB RAM and 32 CPU cores.
# You can skip this step and use your own Seurat object instead
```

### 1. Circlize plot to visualize cell clustering and meta data
```
###Check and see the meta data info on your Seurat object
colnames(iri.integrated@meta.data)  

###Prepare data for ploting
circ_data <- prepare_circlize_data(iri.integrated, scale = 0.8 )
set.seed(1234)
cluster_colors<-rand_color(length(levels(iri.integrated)))
group_colors<-rand_color(length(table(iri.integrated$Group)))
rep_colors<-group_colors<-rand_color(length(table(iri.integrated$orig.ident)))

###plot and save figures
png(filename =  'circlize_plot.png', width = 6, height = 6,units = 'in', res = 300)
plot_circlize(circ_data,do.label = T, pt.size = 0.01, col.use = cluster_colors ,bg.color = 'white', kde2d.n = 200)
add_track(circ_data, group = "Group", colors = group_colors) ## can change it to one of the columns in the meta data of your seurat object
add_track(circ_data, group = "orig.ident",colors = rep_colors) ## can change it to one of the columns in the meta data of your seurat object
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/circlize_plot.png) <br />

### 2. Dotplot to show gene expression across groups
Here is an example to use plot1cell to show one gene expression across different cell types breakdown by group.
```
png(filename =  'dotplot_single.png', width = 4, height = 6,units = 'in', res = 100)
complex_dotplot_single(seu_obj = iri.integrated, feature = "Havcr1",groupby = "Group")
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/dotplot_single.png) <br />
plot1cell allows visualization of multiple genes in dotplot format too. Here is an example.
```
png(filename =  'dotplot_multiple.png', width = 10, height = 4,units = 'in', res = 300)
complex_dotplot_multiple(seu_obj = iri.integrated, features = c("Slc34a1","Slc7a13","Havcr1","Krt20","Vcam1"),groupby = "Group", celltypes = c("PTS1" ,   "PTS2"  ,  "PTS3"  ,  "NewPT1" , "NewPT2"))
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/dotplot_multiple.png) <br />

### 3. Violin plot to show gene expression across groups
#### One gene/one group violin plot:
```
png(filename =  'vlnplot_single.png', width = 4, height = 6,units = 'in', res = 100)
complex_vlnplot_single(iri.integrated, feature = "Havcr1", groups = "Group",celltypes   = c("PTS1" ,   "PTS2"  ,  "PTS3"  ,  "NewPT1" , "NewPT2"))
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/vlnplot_single.png) <br />

#### One gene/multiple groups violin plot:
```
png(filename =  'vlnplot_multiple.png', width = 8, height = 6,units = 'in', res = 300)
complex_vlnplot_single(iri.integrated, feature = "Havcr1", groups = c("Group","Replicates"),celltypes   = c("PTS1" ,   "PTS2"  ,  "PTS3"  ,  "NewPT1" , "NewPT2"))
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/vlnplot_multiple.png) <br />

Note that the Replicates group here is for demo purpose. This is not the mouse ID as reported in our original paper.

#### Multiple genes/multiple groups.
The violin plot will look too messy in this scenario so it is not included in plot1cell. It is highly recommended to use the complex_dot_plot instead. <br />
For scRNA-seq studies with higher complexity, the complex_vlnplot_single function also allows split the group by another group in the meta data with the argument "split.by". Since the demo dataset doesn't have this complexity, examples are not included here. Users can refer to our recent DKD dataset with multiple treatments/two timepoints (the PCT violin graph in <a href="https://humphreyslab.com/SingleCell/">K.I.T.</a>)

### 4. Umap geneplot across groups
```
png(filename =  'geneplot_umap.png', width = 4, height = 6,units = 'in', res = 100)
complex_featureplot(iri.integrated, features = c("Havcr1","Slc34a1"), group = "Group", select = c("Control","12hours","6weeks"), order = T)
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/geneplot_umap.png) <br />

### 5. ComplexHeatmap to show unique genes across groups
plot1cell can directly identify the condition specific genes in a selected cell type and plot those genes using ComplexHeatmap. An example is shown below:
```
iri.integrated$Group2<-mapvalues(iri.integrated$Group, from = c("Control", "4hours",  "12hours", "2days",   "14days" , "6weeks" ),
to = c("Ctrl","Hr4","Hr12","Day2", "Day14","Wk6"))
iri.integrated$Group2<-factor(iri.integrated$Group2, levels = c("Ctrl","Hr4","Hr12","Day2", "Day14","Wk6"))
png(filename =  'heatmap_group.png', width = 4, height = 8,units = 'in', res = 100)
complex_heatmap_group(seu_obj = iri.integrated, celltype = "NewPT2", group = "Group2",gene_highlight = c("Slc22a28","Vcam1","Krt20","Havcr1"))
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/heatmap_group.png) <br />

### 6. Other ploting functions
There are other functions for plotting/data processing in plot1cell. Many more functions will be added in the future package development. For questions, please raise an issue in this github page or contact <a href="https://humphreyslab.com">TheHumphreysLab</a>. 
