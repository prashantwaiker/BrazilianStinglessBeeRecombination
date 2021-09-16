# Brazilian Stingless Bee (Frieseomelita varia) Recombination map

This repository contains data and scripts used to construct high-density linkage map for frieseomelitta varia (Waiker et al 2021, BMC Genomics). The summarized workflow was as follows:

## VCF file filtering on Bash
### Filerting raw .VCF files and ecxtraction of genotypes
(1) SNPs filtered for at least 50% missing data, minimum quality score of 30 and Minor allele frequency of 3:
```
vcftools --vcf fv.vcf --max-missing 0.5 --minQ 30 --minDP 6 --mac 3  --recode --recode-INFO-all --out fv_filter1
```

(2) Genotype information extraction from filtered VCF files:
```
vcftools --vcf fv_filtered.recode.vcf --extract -FORMAT -info GT
```

## Data filetring on Macro VBA
### Multiple rounds of data filetring were applied on dataset containing genotype information. 
Some example codes for filetring process are given below which can be used depending on what you are trying to achieve (the dataset contains SNPs in rows and Offsprings in columns). A apostrophe (') prefix signifies a comment in the code:

(1) highlight the rows that have same data in first four columns:

```
Sub samedata()						'name of macro code
LastRow = Cells(Rows.Count, "A").End(xlUp).Row  	'define last row containing data in column A
LastCol = Cells(1, Columns.Count).End(xlToLeft).Column  ' define last column containing data for row 1

For yrow = 2 To LastRow					'for loop that runs from row 2 to last row
If Cells(yrow, "B") = Cells(yrow, "C") And Cells(yrow, "C") = Cells(yrow, "D") And Cells(yrow, "D") = Cells(yrow, "E") And Cells(yrow, "B") = Cells(yrow, "E") Then   ' If-then statement to compare values between different cells
    If Cells(yrow, "F") = Cells(yrow, "G") And Cells(yrow, "G") = Cells(yrow, "H") And Cells(yrow, "H") = Cells(yrow, "I") And Cells(yrow, "I") = Cells(yrow, "F") Then
        Cells(yrow, 1).EntireRow.Interior.ColorIndex = 3			'Highlight entire row with a specific color if above criteria meets
    End If
End If					' logical statement to instruct the program to stop if statement
Next yrow				'for loop goes to next row
End Sub					'end macro
```

(2) Copy entire rows if a cell has a specific color and paste it to new excel sheet
```
Sub Sortbycolor()				'macro name
Dim Sourcews As Worksheet			' defining worksheets as variable 'Worksheet'
Dim Destws As Worksheet
LastRow = Cells(Rows.Count, "A").End(xlUp).Row

Set Sourcews = ActiveSheet			'Setting current worksheet as active
Set Destws = Worksheets.Add			'create a new sheet to paste the values

zrow = 2
For yrow = 1 To LastRow
    If Sourcews.Cells(yrow, 1).Interior.Color = vbRed Then	'If then statement to check if a cell has a specific background color
    Sourcews.Cells(yrow, 1).EntireRow.Copy Destws.Cells(zrow, 1) ' If criteria meets then paste the row to destinamtion worksheet
    zrow = zrow + 1						'increasing the value of destination row number to paste next value in empty row
    End If
Next yrow
    
End Sub

```

(3) #Compare the markers from a list and highlight the row

```
Sub compareandhighlightmarkers()
x = 1
Do While Cells(x, 1) <> ""      'List of markers that need to be tested
    Sheets(1).Select		'Sheet containing marker number --Markers to delete
    temp = Cells(x, 1)		' temporary variable to hold marker information
    Sheets(2).Select
    Lastrow = Cells(Rows.Count, 1).End(xlUp).Row
    For zrow = 2 To Lastrow
        If Cells(zrow, 1) = temp Then	'compares if value matches
            Cells(zrow, 1).EntireRow.Interior.ColorIndex = 3  'highlight the row if condition meets
        End If
    Next zrow
x = x + 1
Loop
End Sub 
 ```

(4) compare a list of SNPs name to a bigger list with map distance and if match then put map distance next to the SNP
```
Sub Addmapdistance()
x = 2
Do While Cells(x, 1) <> ""    'first column has the SNP list with ID
    temp = Cells(x, 1)
    Lastrow = Cells(Rows.Count, 5).End(xlUp).Row    'row with all markers in col 5
    For zrow = 2 To Lastrow
        If Cells(zrow, 5) = temp Then
            Cells(zrow, 7) = Cells(x, 2)
        End If
    Next zrow
x = x + 1
Loop
End Sub
```
(5) Check file and highlight if the value of a column is lesser than a threshold value
```
Sub Highlight_rowbycondition() 'Highlight rows if cell has certain condition (Conditional formatting)
Lastrow = Cells(Rows.Count, 5).End(xlUp).Row    'row with all markers in col 5 - the other list which does not have an ID
    For zrow = 1 To Lastrow
        If Cells(zrow, 6) < 150 Then
            Cells(zrow, 6).EntireRow.Interior.ColorIndex = 3
        End If
    Next zrow
End Sub
```

## Linkage map construction using R/QTL R package steps

(1) Installing RQTL package
```
install.packages("qtl", dependencies=TRUE)
```
(2) Loading library in R
```
library(qtl)
```

(3) Setting Working directory
```
setwd("C:/fakepath")
```

(4)Setting parameters to print maximum lines in R output window
```
options(max.print=1000000)
```
(5) Loading the CSV genotype file in R where genotypes are either coded as 'A' or 'H' for homozygous and heterozygous
```
fv<- read.cross("csvr", "C:/fakepath", "FV_inputfile.csv")
```
(6) Plotting missing data to inspect any severe missingness
```
plotMissing(fv, main="Missing data for FV markers")
```
(7) Find duplicate markers in vector dupmar
```
dupmar <- findDupMarkers(fv, exact.only = F) 
```
(8)drop duplicate markers
```
fv <- drop.markers(fv, unlist(dupmar))
```
(9) write new file with no duplicate marker
```
write.cross(fv, format='csvr',filestem = "nodupLG1")
```
(10) check new numbers of markers (optional confirmatory step)
```
totmar(fv)
```
(11) estimating recombination fraction between each marker pait and calculate LOD score for a test of rf=0.5
```
fv<- est.rf(fv)
```
(12) Inspecting list markers for potentially switched allels
```
checkAlleles(fv, threshold=5) 
```
(13) Plotting marker pairs for potentially switched alleles
```
rf <- pull.rf(fv)
lod <- pull.rf(fv, what="lod")
plot(as.numeric(rf), as.numeric(lod), xlab="Recombination fraction", ylab="LOD score")
```
(14) Inferring linkage groups
```
lg <- formLinkageGroups(fv, max.rf=0.25, min.lod= 5)
table(lg[,2])
```
(15) reorganizing inferred linkage groups
```
lg<- formLinkageGroups(fv, max.rf=0.25, min.lod=5, reorgMarkers = T)
```
(16) Pull Recombination fraction matrix to inspect potential connections between markers of linkage groups
```
RFmatrix_endmarkers <- pull.rf(fv, what="lod")
write.table(RFmatrix_endmarkers, 'LODmatrix_endmarkers.csv', sep='\t')
```
(17) Ordering markers within linkage groups and saving as a csv file on local computer (repeat code for each linkage group)
```
fv1<- orderMarkers(lg, chr = 1, error.prob = 0.001)
sink(file = "FV_LG1.csv")
pull.map(fv1,chr = 1)
sink()
```
(18) Drop one suspected bad marker at a time to inspect if gaps are being created by that marker
```
dropone <- droponemarker(lg, error.prob=0.001)
par(mfrow=c(2,1))
plot(dropone, lod=1, ylim=c(-100,0))
plot(dropone, lod=2, ylab="Change in chromosome length")
par(mfrow=c(1,1))
```

(19) Tryallpositions() to try an additional marker if it fits in already created linakge map
```
tryallpositions(lg,"SNPxx", chr = 1, error.prob = 0.001)
```
(20) Loop code to try multiple markers for tryallpositions using a list of test markers
```
markers<- scan(file ="listofextramarkers.csv", character(), quote = "", skip = 1)
output <- vector("list", length(markers))
names(output) <- markers
for(i in seq_along(markers)) {
  output[[i]] <- tryallpositions(lg, markers[i], error.prob = 0.001)    #specify an LG using argument chr=x to avoid intensive never ending computation
}
sink(file="Trymarker_allLG.csv")
output
sink()
```

(21) Plotting linkage groups using library LinkageMapView
```
library(LinkageMapView)
outfile = file.path("C:/fakepath", "fv1.pdf")
lmv.linkage.plot(fv1, outfile)   #repeat for each linkage group
```
(22) Plotting all linkage groups in a single plot along with marker density
```
maxpos <- 310   # draw tickmarks at each cM from 0 to largest position of linkage groups to be drawn
at.axis <- seq(0, maxpos)
axlab <- vector()
for (lab in 0:maxpos) {
  if (!lab %% 50) {
    axlab <- c(axlab, lab)
  }
  else {
    axlab <- c(axlab, NA)
  }
}
outfile = file.path("C:/fakepath", "linkagemapwithdensity.pdf")
lmv.linkage.plot(fv, outfile, denmap = TRUE, cex.axis = 2,cex.lgtitle = 4 ,at.axis = at.axis,labels.axis = axlab, pdf.height = 16)
```

## Phylogenetic analysis to compare recdombination rates of different social insect species using R package 'Phytools'
```
library(phytools)   #package to create phylogeny figure
library(ggplot2)
tree2<-read.tree("FileS3.tre").       'input file is a .tre file in Newick format that contains divergence data
x2<-read.csv("FileS4.csv",header=TRUE,row.names=1) ' input file is a .csv file that conatins name of species and recombination rates
x2<-setNames(x2[,1],rownames(x2))
obj<-contMap(tree2,x2,plot=FALSE, method= "fastAnc")
obj<-setMap(obj,invert=TRUE)
plot(obj,fsize=c(1,0.8),outline=FALSE,lwd=c(6,7),leg.txt="Recombination rate (cM/Mb)")
```
## Supplementary analyses codes
Correlation analysis Physical length and genetic length for each LG
```
'q-q plot for normality assumption
ggqqplot(my_data$Contigs_length_Mb, ylab = "Physical length")
ggqqplot(my_data$Avg.RR, ylab = "Rec. rates")

'Normality check statistical test
shapiro.test(my_data$Contigs_length_Mb)
shapiro.test(my_data$Avg.RR)

'plotting correlation graph
library("ggpubr")
my_data <- read.csv("Map_summary.csv")
ggscatter(my_data, x = "Contigs_length_Mb", y = "Avg.RR", 
          add = "reg.line", conf.int = TRUE, 
          cor.coef = TRUE, cor.method = "spearman",
          xlab = "Physical length of LG (Mb)", ylab = "Average Recombination rate of LG (cM/Mb)")

'Rho statistic for spearman correlation test
rho<-cor.test(my_data$Contigs_length_Mb, my_data$Avg.RR,  method = "spearman")
rho
```
