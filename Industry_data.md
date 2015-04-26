# Processing of industry data from questionnaire survey regarding vessel and gear specifications

# Introduction #

The R code to proceed the gear data obtained from the questionnaire survey is given as illustration. Equivalent coding is applied for OT, TBB, DS and DRB gear data (with some specifics for each of them). The final product is a table of parameters (estimates\_for\_gear\_param\_per\_metier.txt) defining the relationships to be used in the BENTHIS WP2 workflow.

This table will be disseminated by mails to partners so that they can incorporate it to the BENTHIS WP2 workflow.

Note that some BENTHIS metiers (e.g. OT\_CRU, etc.) are defined on the way which supposes that each EFLALO metier (e.g. OTB\_CRU\_70-89\_2\_35) should be linked to them in advance of using the table in the BENTHIS WP2 workflow. To proceed, it is basically one line of code that each partner should incorporate in their own workflow to convert the eflalo metier into the BENTHIS metier categorization. This could be for example levels(tacsatp$LE\_MET) <- c('OT\_CRU', 'OT\_DEM', 'OT\_MIX', etc.).


# Details #


Hereafter, the R code to incorporate the parameter table (estimates\_for\_gear\_param\_per\_metier.txt) into the BENTHIS WP2 workflow which assigns a gear width to each metier of each vessel recorded in the tacsatp merged data, further accounting for uncertainties by defining lower and upper estimates of the gear width (or length of the rope for seiners):


```



#- Make selection for gears where you already have gear width and which not
ctry <- "XXX"
if(ctry=="NLD"){
tacsatpWithWidth      <- subset(tacsatp, LE_GEAR %in% c("DRB","TBB"))
tacsatpNonWidth       <- subset(tacsatp,!LE_GEAR %in% c("DRB","TBB"))
} else{
tacsatpWithWidth      <- NULL
tacsatpNonWidth       <- tacsatp
}

# MERGE WITH GEAR WIDTH
# CAUTION: the LE_MET should be consistent with those described in the below table!
# if not then redefine them BEFORE making this step!
# import the param table obtained from the industry_data R analyses
gear_param_per_metier       <- read.table(file=file.path(dataPath, "estimates_for_gear_param_per_metier.txt"))

GearWidth                   <- tacsatpNonWidth[!duplicated(data.frame(tacsatpNonWidth$VE_REF,tacsatpNonWidth$LE_MET)), ]
GearWidth                   <- GearWidth[,c('VE_REF','LE_MET','VE_KW', 'VE_LEN') ]
GearWidth$GEAR_WIDTH        <- NA
GearWidth$GEAR_WIDTH_LOWER  <- NA
GearWidth$GEAR_WIDTH_UPPER  <- NA
for (i in 1:nrow(GearWidth)) { # brute force...
kW      <- GearWidth$VE_KW[i]
LOA     <- GearWidth$VE_LEN[i]
this    <- gear_param_per_metier[gear_param_per_metier$a_metier==GearWidth$LE_MET[i],]
a <- NULL ; b <- NULL
a       <- this[this$param=='a', 'Estimate']
b       <- this[this$param=='b', 'Estimate']
GearWidth[i,"GEAR_WIDTH"]  <-   eval(parse(text= as.character(this[1, 'equ']))) / 1000 # converted in km
a       <- this[this$param=='a', 'Estimate'] +2*this[this$param=='a', 'Std..Error']
b       <- this[this$param=='b', 'Estimate'] +2*this[this$param=='b', 'Std..Error']
GearWidth[i,"GEAR_WIDTH_UPPER"]  <-  eval(parse(text= as.character(this[1, 'equ']))) / 1000 # converted in km
a       <- this[this$param=='a', 'Estimate'] -2*this[this$param=='a', 'Std..Error']
b       <- this[this$param=='b', 'Estimate'] -2*this[this$param=='b', 'Std..Error']
GearWidth[i,"GEAR_WIDTH_LOWER"]  <-  eval(parse(text= as.character(this[1, 'equ']))) / 1000 # converted in km
}
save(GearWidth, file=file.path(outPath,a_year,"gearWidth.RData"))
load(file.path(outPath,a_year,"gearWidth.RData"))
tacsatpNonWidth                    <- merge(tacsatpNonWidth, GearWidth,by=c("VE_REF","LE_MET","VE_KW","VE_LEN"),
all.x=T,all.y=F)

#- Combine tacsat with NonWidth and With
tacsatpWithWidth                   <- cbind(tacsatpWithWidth,
cbind(GEAR_WIDTH        = tacsatpWithWidth$LE_WIDTH / 1000,
GEAR_WIDTH_LOWER  = tacsatpWithWidth$LE_WIDTH / 1000,
GEAR_WIDTH_UPPER  = tacsatpWithWidth$LE_WIDTH / 1000))

tacsatp                            <- rbindTacsat(tacsatpWithWidth,tacsatpNonWidth)
save(tacsatp,   file=file.path(outPath,a_year,"tacsatMergedWidth.RData"))


```

The table of parameters (relationships to predict the gear width per BENTHIS metier from the kW or LOA) has been built from the following R code, after collating the outcomes of the analysis split by OT, TBB, DRB and DS for Otter trawl, beam trawl, dredge and seine respectively.

```


# plot (with ggplot2)
library(ggplot2)

# paths
dataPath  <- file.path("C:", "BENTHIS", "data_gear_spec_questionnaire")
outPath   <- file.path("C:", "BENTHIS", "data_gear_spec_questionnaire")



##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-----------------OT------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------

# read file
ind_DK  <- read.table(file= file.path(dataPath, 'OT', 'OT_DK_31012014.csv'), sep=";", header=TRUE )
ind_IRE <- read.table(file= file.path(dataPath, 'OT', 'OT_IRE_28012014.csv'), sep=";", header=TRUE )
ind_SCO <- read.table(file= file.path(dataPath, 'OT', 'OT_SCO_28012014.csv'), sep=";", header=TRUE )
ind_SWE <- read.table(file= file.path(dataPath, 'OT', 'OT_SWE_28012014.csv'), sep=";", header=TRUE )
ind_NL  <- read.table(file= file.path(dataPath, 'OT', 'OT_NL_28012014.csv'), sep=";", header=TRUE )
ind_TUR <- read.table(file= file.path(dataPath, 'OT', 'OT_TUR_28012014.csv'), sep=";", header=TRUE )
ind_MED <- read.table(file= file.path(dataPath, 'OT', 'OT_MED_28012014.csv'), sep=";", header=TRUE ) # italian & spanish data
ind_NOR <- read.table(file= file.path(dataPath, 'OT', 'OT_NOR_28012014.csv'), sep=";", header=TRUE )
ind_BEL <- read.table(file= file.path(dataPath, 'OT', 'OT_BEL_28012014.csv'), sep=";", header=TRUE )
ind_FRA <- read.table(file= file.path(dataPath, 'OT', 'OT_FRA_28012014.csv'), sep=";", header=TRUE )
ind_GRE <- read.table(file= file.path(dataPath, 'OT', 'OT_GRE_28012014.csv'), sep=";", header=TRUE )

# collate
cols <- c('Anonymous.vessel_ID','Variable.name','Variable','Value')
ind  <- rbind.data.frame (
cbind(country="DK", ind_DK [, cols]),
cbind(country="IRE", ind_IRE [, cols]),
cbind(country="SCO", ind_SCO [, cols]),
cbind(country="SWE", ind_SWE [, cols]),
cbind(country="NL", ind_NL [, cols]),
cbind(country="TUR", ind_TUR [, cols]),
cbind(country="MED", ind_MED [, cols]),
cbind(country="NOR", ind_NOR [, cols]),
cbind(country="BEL", ind_BEL [, cols]),
cbind(country="FRA", ind_FRA [, cols]),
cbind(country="GRE", ind_GRE [, cols])
)


# explore
head(ind [ind$Variable.name=="Trawl_model",])
levels(ind$Variable.name)
an <- function(x) as.numeric(as.character(x))

# check complete case for some variables (which need to be strictly informed together for a given vessel)
vids_Clp <- unique(as.character(ind [ind$Variable.name=="Clump_weight", "Anonymous.vessel_ID"]))
vids_DoW <- unique(as.character(ind [ind$Variable.name=="Door_weight", "Anonymous.vessel_ID"]))
vids_to_be_removed <- vids_Clp[!(vids_Clp %in% vids_DoW)]
vids_to_be_removed <- c(vids_to_be_removed, vids_DoW[!(vids_DoW %in% vids_Clp)])
#=> if any, then remove the concerned vessel.
ind <- ind [!ind$Anonymous.vessel_ID %in% vids_to_be_removed, ]




CT   <- ind [ind$Variable.name=="Consumption_trawling", "Value"]
CS   <- ind [ind$Variable.name=="Consumption_steaming", "Value"]
Str  <- ind [ind$Variable.name=="Speed_trawling", "Value"]
kW   <- ind [ind$Variable.name=="Vessel_kW", "Value"]
LOA  <- ind [ind$Variable.name=="Vessel_LOA", "Value"]
DoS  <- ind [ind$Variable.name=="Door_spread", "Value"]
DoW  <- ind [ind$Variable.name=="Door_weight", "Value"]
DoN  <- ind [ind$Variable.name=="Door_number", "Value"]
OcW  <- ind [ind$Variable.name=="Otherchain_weight", "Value"]
OcN  <- ind [ind$Variable.name=="Otherchain_number", "Value"]
TiW  <- ind [ind$Variable.name=="Ticklerchain_weight", "Value"]
TiN  <- ind [ind$Variable.name=="Ticklerchain_number", "Value"]
Clp  <- ind [ind$Variable.name=="Clump_weight", "Value"]
GrW  <- ind [ind$Variable.name=="Groundgear_weight", "Value"]
GrL  <- ind [ind$Variable.name=="Groundgear_length", "Value"]
sps  <- ind [ind$Variable.name=="Targetspecies_single", "Value"]
sp1  <- ind [ind$Variable.name=="Primarytarget_mixed", "Value"]
sp2  <- ind [ind$Variable.name=="Secondarytarget_mixed", "Value"]
sp3  <- ind [ind$Variable.name=="Thirdtarget_mixed", "Value"]
bt   <- ind [ind$Variable.name=="Bottom_type", "Value"]
nbt  <- ind [ind$Variable.name=="Trawl_number", "Value"]
sptr <- ind [ind$Variable.name=="Speed_trawling", "Value"]
spst <- ind [ind$Variable.name=="Speed_steaming", "Value"]
ctry <- ind [ind$Variable.name=="Trawl_number", "country"]
area <- ind [ind$Variable.name=="Fishing_area", "Value"]
mesh <- ind [ind$Variable.name=="Codend_meshsize", "Value"]
swee <- ind [ind$Variable.name=="Sweerp_length", "Value"]


# intermediate calculation
# total gear weight
dd    <- rbind(c(an(DoW) *  an(DoN) + an(GrW) * an(nbt)), an(Clp), c(an(OcW) * an(OcN)))
TotGW <- apply(dd,2, sum, na.rm=TRUE) # DoW and GrW need complete cases here.
TotGW[is.na(dd[1,])] <- NA

# then, collate:
df1  <- cbind.data.frame(ctry,CT, CS, sptr, spst, kW, LOA, DoS, DoW, DoN, GrW, nbt, Clp, OcW, OcN, TotGW, GrL, sps, sp1, sp2, sp3, bt, area, mesh, swee)



# look at a potential proxy in case the door spread is not informed....
plot(an(df1$DoS),an(df1$swee), col=df1$ctry, pch=16)



# shortcut to retrieve a given Value
idx <- ind [ind$Variable.name=="Fishing_area", "Value"]==2.2
dd  <- ind [ind$Variable.name=="Fishing_area", ]
dd[idx,]

idx <- ind [ind$Variable.name=="Door_spread", "Value"]==0
dd  <- ind [ind$Variable.name=="Door_spread", ]
dd[idx,]


# refactor some variables (if needed)
df1$LOA_class     <- cut(an(df1$LOA), breaks=c(0,12,18,24,40,100))
df1$sptr          <- cut(an(df1$sptr), breaks= seq(0,10,0.75))   # trawling speed
df1$mesh_class    <- cut(an(df1$mesh), breaks= c(0,90,220))   # codend mesh size
df1$GrW_class     <- cut(an(df1$GrW), breaks= seq(0,1600, 200) )   # codend mesh size


# area coding
df1$area <- as.character(df1$area) # init
df1[df1$area %in% c('IIIa', 'IIIan', 'IIIas', 'IIIast'), 'area'] <- 'kask'
df1[df1$area %in% c('II','IV', 'IVa', 'IVb', 'IVbc','IVc', 'Ivb'), 'area'] <- 'nsea'
df1[df1$area %in% c('IIIc', 'IIId', '25'), 'area'] <- 'bsea'
df1[df1$area %in% c('VII','Vb1','VIa'), 'area'] <- 'csea'
#df1[df1$area %in% c('2.1', '1.3', '2.2', '1.2','0','1.1',''), 'area'] <- 'msea'
df1[df1$area %in% c('0',''), 'area'] <- 'ni'
df1[df1$area %in% c('Black Sea, Samsun Shelf Area (SSA)'), 'area'] <- 'blsea'
df1$area <- as.factor(df1$area)



# DCF metier coding
df1$metier <- as.character(df1$sps) # init
df1[df1$metier %in% c('NEP ','NEP','PRA','Nephrops','Nephrops trawl',
'TGS', 'ARA','DPS' ), 'metier'] <- 'OT_CRU'
df1[df1$metier %in% c('COD','PLE','SOL', 'LEM', 'WHG', 'WHI', 'POK',
'PDS','HAD','had','HKE','MON', 'MUT',
'NOP'), 'metier'] <- 'OT_DMF'
df1[df1$metier %in% c('SAN','SPR','CAP'), 'metier'] <- 'OT_SPF'
df1[df1$metier %in% c('NR', "NI",'0','','ni'), 'metier'] <- 'OT_MIX'
df1$metier <- as.factor(df1$metier)




# species assemblage coding
df1$sp1_type <- as.character(df1$sp1) # init
df1[df1$sp1_type=="NR" | df1$sp1_type=="ni" | df1$sp1_type=="NI" | df1$sp1_type=="" | df1$sp1_type=="0", 'sp1_type'] <-
as.character(df1[df1$sp1_type=="NR"  | df1$sp1_type=="ni" | df1$sp1_type=="NI" | df1$sp1_type=="" | df1$sp1_type=="0", 'sps'])
df1[df1$sp1_type %in% c('PLE', 'SOL', 'LEM', 'MON'), 'sp1_type' ]        <- "Benthic"
df1[df1$sp1_type %in% c('NEP','NEP ',  'PRA', 'DPS',
'Nephrops', 'Nephrops trawl',
'ARA',  'TGS', 'CSH'), 'sp1_type' ]              <- "Crustacean"
df1[df1$sp1_type %in% c('COD', 'POK', 'WHI', 'HAD', 'had',
'WHG',  'HKE', 'MUT','WHB', 'PDS'), 'sp1_type' ] <- "Benthic_Pelagic"
df1[df1$sp1_type %in% c('NOP',  'SAN', 'SPR','CAP'), 'sp1_type' ]        <- "Pelagic"
df1[df1$sp1_type=="NR" | df1$sp1_type=="ni" | df1$sp1_type=="NI" | df1$sp1_type=="" | df1$sp1_type=="0", 'sp1_type']  <- 'NI' #
df1$sp1_type <- as.factor(df1$sp1_type) # init


# level 6 metier with mesh
df1$metier2  <- as.character(df1$metier) # init
df1[df1$metier %in% 'OT_MIX', 'metier2' ]        <- paste(df1[df1$metier %in% 'OT_MIX','metier'], df1[df1$metier %in% 'OT_MIX','mesh_class'], sep="_")
df1$metier2  <- as.factor(df1$metier2)


# look at this....
#hist(an((df1[df1$metier=="OT_MIX", "mesh"])), nclass=50)
meshes <- table(an((df1[df1$metier=="OT_MIX", "mesh"])))


# level 6 metier for MIX with either area (in MED) or mesh (otherwise)
df1$metier3  <- as.character(df1$metier) # init
df1[df1$metier %in% 'OT_MIX' & !df1$ctry %in% 'MED', 'metier3' ]        <-
paste(df1[df1$metier %in% 'OT_MIX'  & !df1$ctry %in% 'MED','metier'], df1[df1$metier %in% 'OT_MIX'  & !df1$ctry %in% 'MED','mesh_class'], sep="_")
df1[df1$metier %in% 'OT_MIX' & df1$ctry %in% 'MED', 'metier3' ]        <-
paste(df1[df1$metier %in% 'OT_MIX'  & df1$ctry %in% 'MED','metier'], df1[df1$metier %in% 'OT_MIX' & df1$ctry %in% 'MED','area'], sep="_")
df1$metier3  <- as.factor(df1$metier3)




# deal with mixed fisheries
df1$met    <- as.character(df1$metier) # init
df1$metier <- as.character(df1$metier) # init
#df1[df1$metier %in% c('OT_MIX') & sp1 %in% c('NEP ','NEP','PRA',
#                                'Nephrops','Nephrops trawl','TGS',
#                                'ARA','DPS'), 'metier'] <- 'OT_MIX_CRU'
df1[df1$metier %in% c('OT_MIX') & df1$sp1 %in% c('NEP ','NEP',
'Nephrops','Nephrops trawl', 'PRA', 'CSH'
), 'metier'] <- 'OT_MIX_NEP'
#df1[df1$metier %in% c('OT_MIX') & df1$sp1 %in% c(), 'metier'] <- 'OT_MIX_PRA'
df1[df1$metier %in% c('OT_MIX') & df1$sp1 %in% c('TGS') &  df1$sp2 %in% c('OCC'), 'metier'] <- 'OT_MIX_TGS_OCC'
df1[df1$metier %in% c('OT_MIX') & df1$sp1 %in% c('TGS') &  df1$sp2 %in% c('CTC'), 'metier'] <- 'OT_MIX_TGS_CTC'
df1[df1$metier %in% c('OT_MIX') & df1$sp1 %in% c('ARA'
), 'metier'] <- 'OT_MIX_ARA'
df1[df1$metier %in% c('OT_MIX') & df1$sp1 %in% c('DPS'
), 'metier'] <- 'OT_MIX_DPS'
df1[df1$metier %in% c('OT_MIX') & df1$sp1 %in% c('PLE','SOL', 'LEM',
'MON', 'MUT') & df1$sp1_type=="Benthic", 'metier'] <- 'OT_MIX_DEM_Benthic'
df1[df1$metier %in% c('OT_MIX') & df1$sp1 %in% c('COD','PLE','SOL', 'LEM',
'WHG', 'WHI', 'POK', 'PDS','HAD','had','HKE',
'MON', 'MUT', 'NOP')& df1$sp1_type=="Benthic_Pelagic", 'metier'] <- 'OT_MIX_DEM_Benthic_Pelagic'

df1[df1$metier %in% c('OT_MIX') & df1$sp1 %in% c('SAN',
'SPR'), 'metier'] <- 'OT_MIX_SPF'
df1$met    <- as.factor(df1$met)
df1$metier <- as.factor(df1$metier)





# bottom type coding
df1$bt <- as.character(df1$bt) # init
df1[df1$bt %in% c('hard', 'mud','Hard', 'Hard/sand', 'hard bottom. gravel. sand',
'hard clay. mud','bedrock, hard bottom.  mud',
'hard bottom. mud',
'hard bottom with empty mussel and calcareous. mud',
'hard bottom with empty mussel and calcareous. mud. sand',
' hard bottom with empty mussel and calcareous, mud',
'bedrock. hard bottom.  mud',
'Hard/send', 'Hard/Sand', 'bedrock.','bedrock. hard bottom mud',  'hard bottom.' ), 'bt']                                        <- 'coarse'
df1[df1$bt %in% c("sandy","Sand", "sand. mud", 'Sandy', '3', 'Sand, gravel', 'sand',
'sand. clay. mud', 'Sand/Clay' ), 'bt'] <- 'sand'
df1[df1$bt %in% c("Clay/mud ", 'Clay/mud', 'Sand/clay', 'Sandy Clay', 'Mud','Sand/mud',
'Clay/mud','clay, mud', 'Clay', '5',
'sand. hard clay. mud', 'muddy', 'mud', 'mud ', 'mud'), 'bt']        <- 'mud'
df1[df1$bt %in% c("mixed", "combination","hard bottom. sand.  clay. mud","gravel. sand. clay. mud",
"gravel. hard bottom. sand. clay. mud", 'sand. clay. mud',
'hard bottom. sand', 'sand. hard. mud',  "hard bottom. gravel. sand. hard clay. mud", "hard. mud",
'hard bottom. sand. mud', 'ni', 'NI'), 'bt']                             <- 'mixed'
df1$bt <- as.factor(df1$bt)



# nb of observations per country
table(df1$ctry)

# nb of observations per metier
table(df1$metier)

# nb of observations per country per metier
table(df1$ctry,df1$metier)

# nb of observations per country per metier
table(df1$ctry,df1$metier3)

##-------------------------------------------------
##-------------------------------------------------
## Door spread vs. kW
##-------------------------------------------------
##-------------------------------------------------

# look at the representativity....
df1$informedDoS <- ifelse(is.na(an(df1$DoS)) | an(df1$DoS)==0,0,1)
table(df1$ctry, df1$informedDoS)

# look at the representativity....
df1$informedarea <- ifelse(as.character(df1$area)=='0',0,1)
table(df1$ctry, df1$informedarea)


# look at the representativity....
table(df1$ctry, df1$metier, df1$informedDoS)

# filter out the 0
df1_DoS <- df1[!is.na(df1$DoS) & df1$DoS!=0,]

# plot DoS
windows(16, 5)
library(ggplot2)
coeff_lm <- coef(lm(an(DoS) ~ an(kW) , data = df1_DoS))
p <- ggplot(data=df1_DoS, aes(x=an(kW),y=an(DoS), color= factor(metier))) +
geom_point(aes(shape  = factor(metier), color= factor(metier))) +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=1,cc=1))
#stat_smooth( method="glm", family="poisson", formula='DoS~kW+metier')
# stat_smooth(method="lm")
update_labels(p, list(x = "kW", y="Door_spread"))
savePlot(file=file.path(outPath, "plot_OT_nls_DoS_vs_kW_per_metier.png"), type="png")

# plot DoS nls fit with area and ctry
library(ggplot2)
#levels(df1_DoS$area) <- 0:length(df1_DoS$area)
coeff_lm <- coef(lm(an(DoS) ~ an(kW) , data = df1))
p <- ggplot(data=df1_DoS, aes(x=an(kW),y=an(DoS))) +
scale_shape_manual(values = 1:length(levels(df1_DoS$area))) +
geom_point(aes(shape  = factor(area), color= factor(ctry)), size=2) +
#stat_smooth(method="lm") +
facet_grid(. ~ metier, scales="free") +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=50,cc=1))
update_labels(p, list(x = "kW", y="Door_spread"))
savePlot(file=file.path(outPath, "plot_OT_nls_DoS_vs_kW_per_area_ctry.png"), type="png")

# plot DoS  nls fit with bottom type and ctry
library(ggplot2)
coeff_lm <- coef(lm(an(DoS) ~ an(kW) , data = df1))
p <- ggplot(data=df1_DoS, aes(x=an(kW),y=an(DoS))) +
geom_point(aes(shape  = factor(bt), color= factor(ctry))) +
#stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
facet_grid(. ~ metier, scales="free") +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=50,cc=1), control=nls.control(maxiter = 300))
update_labels(p, list(x = "kW", y="Door_spread"))
savePlot(file=file.path(outPath, "plot_OT_nls_DoS_vs_kW_per_bt_ctry.png"), type="png")


# plot DoS  nls fit with sp1_type and ctry
library(ggplot2)
coeff_lm <- coef(lm(an(DoS) ~ an(kW) , data = df1))
p <- ggplot(data=df1_DoS, aes(x=an(kW),y=an(DoS))) +
geom_point(aes(shape  = factor(sp1_type), color= factor(ctry))) +
#stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
facet_grid(. ~ metier, scales="free") +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=50,cc=1))
update_labels(p, list(x = "kW", y="Door_spread"))
savePlot(file=file.path(outPath, "plot_OT_nls_DoS_vs_kW_per_sp1_type_ctry.png"), type="png")


# plot DoS  nls fit with nb of trawls and ctry
library(ggplot2)
coeff_lm <- coef(lm(an(DoS) ~ an(kW) , data = df1))
p <- ggplot(data=df1_DoS, aes(x=an(kW),y=an(DoS))) +
geom_point(aes(shape  = factor(nbt), color= factor(ctry))) +
#stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
facet_grid(. ~ metier, scales="free") +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=50,cc=1))
update_labels(p, list(x = "kW", y="Door_spread"))
savePlot(file=file.path(outPath, "plot_OT_nls_DoS_vs_kW_per_nbtrawls_ctry.png"), type="png")

# plot DoS  nls fit with trawling speed and ctry
library(ggplot2)
coeff_lm <- coef(lm(an(DoS) ~ an(kW) , data = df1))
p <- ggplot(data=df1_DoS, aes(x=an(kW),y=an(DoS))) +
geom_point(aes(shape  = factor(sptr), color= factor(ctry))) +
#stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
facet_grid(. ~ metier, scales="free") +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=50,cc=1))
update_labels(p, list(x = "kW", y="Door_spread"))
savePlot(file=file.path(outPath, "plot_OT_nls_DoS_vs_kW_per_speedtr_ctry.png"), type="png")


# plot DoS  nls fit with mesh and ctry
library(ggplot2)
coeff_lm <- coef(lm(an(DoS) ~ an(kW) , data = df1))
p <- ggplot(data=df1_DoS, aes(x=an(kW),y=an(DoS))) +
geom_point(aes(shape  = factor(mesh), color= factor(ctry))) +
#stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
facet_grid(. ~ metier, scales="free") +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=50,cc=1))
#stat_smooth( method="nls", formula='y~k*sqrt(x)', se=FALSE, start=list(k=50))
update_labels(p, list(x = "kW", y="Door_spread"))
savePlot(file=file.path(outPath, "plot_OT_nls_DoS_vs_kW_per_mesh_ctry.png"), type="png")

# plot DoS  nls fit with metier level6 and metier and  ctry
library(ggplot2)
coeff_lm <- coef(lm(an(DoS) ~ an(kW) , data = df1))
p <- ggplot(data=df1_DoS, aes(x=an(kW),y=an(DoS))) +
scale_shape_manual(values = 1:length(levels(df1_DoS$metier))) +
geom_point(aes(shape  = factor(metier), color= factor(ctry))) +
#stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
facet_grid(. ~ metier2, scales="free") +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=50,cc=1))
#stat_smooth( method="nls", formula='y~k*sqrt(x)', se=FALSE, start=list(k=50))
update_labels(p, list(x = "kW", y="Door_spread"))
savePlot(file=file.path(outPath, "plot_OT_nls_DoS_vs_kW_per_mesh_ctry.png"), type="png")

# plot DoS  nls fit with metier-area or metier-mesh and metier-target and  ctry
library(ggplot2)
coeff_lm <- coef(lm(an(DoS) ~ an(kW) , data = df1))
p <- ggplot(data=df1_DoS, aes(x=an(kW),y=an(DoS))) +
scale_shape_manual(values = 1:length(levels(df1_DoS$metier))) +
geom_point(aes(shape  = factor(metier), color= factor(ctry))) +
#stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
facet_grid(. ~ metier3, scales="free") +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=50,cc=1))
#stat_smooth( method="nls", formula='y~k*sqrt(x)', se=FALSE, start=list(k=50))
update_labels(p, list(x = "kW", y="Door_spread"))
savePlot(file=file.path(outPath, "plot_OT_nls_DoS_vs_kW_per_mesh_ctry.png"), type="png")

# plot DoS  nls fit with metier-area or metier-mesh and metier-target and  ctry
library(ggplot2)
coeff_lm <- coef(lm(an(DoS) ~ an(kW) , data = df1_DoS))
p <- ggplot(data=df1_DoS, aes(x=an(kW),y=an(DoS))) +
scale_shape_manual(values = 1:length(levels(df1_DoS$metier))) +
geom_point(aes(shape  = factor(metier), color= factor(ctry))) +
#stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
facet_grid(. ~ met, scales="free") +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=50,cc=1))
#stat_smooth( method="nls", formula='y~k*sqrt(x)', se=FALSE, start=list(k=50))
update_labels(p, list(x = "kW", y="Door_spread"))
savePlot(file=file.path(outPath, "plot_OT_nls_DoS_vs_kW_per_simple_met_ctry.png"), type="png")





##-------------------------------------------------
##-------------------------------------------------
##  Door spread vs. LOA
##-------------------------------------------------
##-------------------------------------------------

# look at the representativity....
df1$informedDoS <- ifelse(is.na(an(df1$DoS)) | an(df1$DoS)==0,0,1)
table(df1$ctry, df1$informedDoS)

# look at the representativity....
df1$informedarea <- ifelse(as.character(df1$area)=='0',0,1)
table(df1$ctry, df1$informedarea)


# look at the representativity....
table(df1$ctry, df1$metier, df1$informedDoS)

# filter out the 0
df1_DoS <- df1[!is.na(df1$DoS) & df1$DoS!=0,]

# plot DoS vs. LOA per area
library(ggplot2)
coeff_lm <- coef(lm(an(DoS) ~ an(LOA) , data = df1))
p <- ggplot(data=df1_DoS, aes(x=an(LOA),y=an(DoS))) +
scale_shape_manual(values = 1:length(levels(df1_DoS$area))) +
geom_point(aes(shape  = factor(area), color= factor(ctry))) +
#stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
#stat_smooth( method="nls", formula='y~a*(x^b)', se=FALSE, start=list(a=50,b=1), data=subset(df1_DoS, metier=="OT_CRU"))
facet_grid(. ~ metier, scales="free") +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=50,cc=1))
update_labels(p, list(x = "LOA", y="Door_spread"))
savePlot(file=file.path(outPath, "plot_OT_nls_DoS_vs_LOA_per_area_ctry.png"), type="png")



##-----------------------------------------
##-----------------------------------------
## Export a table of parameters
## for the Benthis vmstools R workflow
## after having made a choice of the most relevant
## categorisation
##-----------------------------------------
##-----------------------------------------

# get the coeffs for DoS~ LOA or kW
coeffs_DoS <- NULL
df1_DoS$DoS     <-  an(df1_DoS$DoS)
df1_DoS$LOA     <-  an(df1_DoS$LOA)
df1_DoS$kW      <-  an(df1_DoS$kW)
df1_DoS_c       <- df1_DoS[!is.na(df1_DoS$LOA) & !is.na(df1_DoS$kW),]  # caution: keep complete cases for a true model comparison...
for (a_metier in c("OT_CRU", "OT_DMF", "OT_MIX", "OT_MIX_ARA", "OT_MIX_DEM_Benthic", "OT_MIX_DEM_Benthic_Pelagic", "OT_MIX_DPS", "OT_MIX_NEP","OT_MIX_TGS_CTC", "OT_MIX_TGS_OCC", "OT_SPF")){
a_nls_kW        <- nls(DoS~a*(kW^b), start=list(a=1,b=1),data=df1_DoS_c[df1_DoS_c$metier==a_metier,])
a_lm_kW         <- nls(DoS~a*kW+b, start=list(a=1,b=1),data=df1_DoS_c[df1_DoS_c$metier==a_metier,])
a_nls_LOA       <- nls(DoS~a*(LOA^b), start=list(a=50,b=1),data=df1_DoS_c[df1_DoS_c$metier==a_metier,])
a_lm_LOA        <- nls(DoS~a*LOA+b, start=list(a=50,b=1),data=df1_DoS_c[df1_DoS_c$metier==a_metier,])

#compare goodness of fit
residualSum    <- anova (a_nls_LOA, a_lm_LOA, a_nls_kW, a_lm_kW)
what_is_chosen <- c('a_nls_LOA','a_lm_LOA','a_nls_kW','a_lm_kW') [which.min(residualSum[,2])]
print(what_is_chosen)

nb_records      <- nrow(df1_DoS[df1_DoS$metier==a_metier,])

# then choose the best model....
# (and re-run on the full dataset)
if(what_is_chosen=="a_nls_LOA") {
a_nls_LOA      <- nls(DoS~a*(LOA^b), start=list(a=50,b=1), data=df1_DoS[df1_DoS$metier==a_metier,])  # redo with all the available data
coeffs_DoS     <- rbind.data.frame (coeffs_DoS,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_nls_LOA)$coeff, equ="DoS=a*(LOA^b)", nb_records= nb_records))
}
if(what_is_chosen=="a_lm_LOA"){
a_lm_LOA        <- nls(DoS~a*LOA+b, start=list(a=50,b=1),data=df1_DoS[df1_DoS$metier==a_metier,])
coeffs_DoS      <- rbind.data.frame (coeffs_DoS,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_lm_LOA)$coeff, equ="DoS=(a*LOA)+b", nb_records= nb_records))
}
if(what_is_chosen=="a_nls_kW"){
a_nls_kW        <- nls(DoS~a*(kW^b), start=list(a=1,b=1),data=df1_DoS[df1_DoS$metier==a_metier,])
coeffs_DoS      <- rbind.data.frame (coeffs_DoS,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_nls_kW)$coeff, equ="DoS=a*(kW^b)", nb_records= nb_records))
}
if(what_is_chosen=="a_lm_kW"){
a_lm_kW         <- nls(DoS~a*kW+b, start=list(a=1,b=1),data=df1_DoS[df1_DoS$metier==a_metier,])
coeffs_DoS      <- rbind.data.frame (coeffs_DoS,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_lm_kW)$coeff, equ="DoS=(a*kW)+b", nb_records= nb_records))
}
}
rownames(coeffs_DoS) <- NULL


# export
write.table(coeffs_DoS, file=file.path(outPath, "estimates_for_OT_nls_DoS_vs_LOA_or_kW_per_metier.txt"))
# => for using this table in the workflow, partners should link each logbooks records to the metier categories found in that table...


##-------------------------------------------------
##-------------------------------------------------
## do the correponding plot
##-------------------------------------------------
##-------------------------------------------------


#png(filename = file.path(outPath, paste("plot_estimates_for_OT_nls_DoS_vs_LOA_or_kW_per_metier.png",sep="")),
#                                   width = 1100, height = 2400,
#                                   units = "px", pointsize = 12,  res=300)   # high resolution plot
windows(7,10)
par(mfrow=c(4,3))
par(oma=c(6,4,1,1))
par(mar=c(4,0,2,1))
df1_DoS$the_colors   <- df1_DoS$ctry
#library(RColorBrewer)
#the_colors <- brewer.pal(11, "Paired")
#levels(df1_DoS$the_colors) <- the_colors[1:length(unique(df1_DoS$the_colors))]
rgb.palette <- colorRampPalette(c("green", "red", "blue"),
space = "Lab")
levels(df1_DoS$the_colors) <- rgb.palette (length(unique(df1_DoS$the_colors)))
df1_DoS$the_colors <- as.character(df1_DoS$the_colors)


count <-1
for(met in c("OT_CRU", "OT_DMF", "OT_MIX", "OT_MIX_ARA", "OT_MIX_DEM_Benthic", "OT_MIX_DEM_Benthic_Pelagic", "OT_MIX_DPS", "OT_MIX_NEP","OT_MIX_TGS_CTC", "OT_MIX_TGS_OCC", "OT_SPF")){
a           <- coeffs_DoS[coeffs_DoS$a_metier==met & coeffs_DoS$param=='a', 'Estimate']
b           <- coeffs_DoS[coeffs_DoS$a_metier==met  & coeffs_DoS$param=='b', 'Estimate']
an_equation <- as.character(coeffs_DoS[coeffs_DoS$a_metier==met,][1,'equ'])
if(length(grep('LOA' , an_equation))>0){
range_LOA <- range(df1_DoS[df1_DoS$metier==met, "LOA",], na.rm=TRUE)
LOA       <- seq(range_LOA[1], range_LOA[2], by=1)
plot(df1_DoS[df1_DoS$metier==met, "LOA",], df1_DoS[df1_DoS$metier==met, "DoS"], pch=16, col=df1_DoS[df1_DoS$metier==met, "the_colors"], xlab="LOA (metre)", ylab="Door spread (metre)", axes=FALSE, ylim=c(0,300))
lines(LOA, eval(parse(text= an_equation)))
}
if(length(grep('kW' , an_equation))>0){
range_kW <- range(df1_DoS[df1_DoS$metier==met, "kW",], na.rm=TRUE)
kW       <- seq(range_kW[1], range_kW[2], by=1)
plot(df1_DoS[df1_DoS$metier==met, "kW",], df1_DoS[df1_DoS$metier==met, "DoS"], pch=16, col=df1_DoS[df1_DoS$metier==met, "the_colors"], xlab="kW", ylab="Door spread (metre)", axes=FALSE, ylim=c(0,300))
lines(kW, eval(parse(text= an_equation)))
}
axis(1)
if(count==1) axis(2, las=2)
box()
title(met, cex=0.7)
count <- count+1
}
plot(0,0,type="n", axes=FALSE, xlab="", ylab="")
legend("topright", legend=unique(df1_DoS$ctry), fill=unique(df1_DoS$the_colors), bty="n", cex=1.2, ncol=2)

mtext(side=2, text="Door spread (metre)", line=1, outer=TRUE)
savePlot(file=file.path(outPath, "plot_estimates_for_OT_nls_DoS_vs_LOA_or_kW_per_metier.png"), type="png")

# dev.off()



##-------------------------------------------------
##-------------------------------------------------
## Gear weight vs. explanatory variables includ. kW
##-------------------------------------------------
##-------------------------------------------------
# look at the representativity....
df1$informedGrW <- ifelse(is.na(an(df1$GrW)) | an(df1$GrW)==0,0,1)
table(df1$ctry, df1$informedGrW)

# look at the representativity....
df1$informedDoW <- ifelse(is.na(an(df1$DoW)) | an(df1$DoW)==0,0,1)
table(df1$ctry, df1$informedDoW)

# look at the representativity....
df1$informedClp <- ifelse(is.na(an(df1$Clp)) | an(df1$Clp)==0,0,1)
table(df1$ctry, df1$informedClp)

# look at the representativity....
df1$informedTotGW <- ifelse(is.na(an(df1$TotGW)) | an(df1$TotGW)==0,0,1)
table(df1$ctry, df1$informedTotGW)

# look at the representativity....
df1$informedarea <- ifelse(as.character(df1$area)=='0',0,1)
table(df1$ctry, df1$informedarea)

# look at the representativity....
df1$informedTotGW <- ifelse(is.na(an(df1$TotGW)) | an(df1$TotGW)==0,0,1)
table(df1$metier, df1$informedTotGW)


# filter out the 0
df1_TotGW <- df1[!is.na(df1$TotGW) & df1$TotGW!=0,]




# plot TotGW fit with met and  ctry
library(ggplot2)
coeff_lm <- coef(lm(an(TotGW) ~ an(kW) , data = df1_TotGW))
p <- ggplot(data=df1_TotGW, aes(x=an(kW),y=an(TotGW))) +
scale_shape_manual(values = 1:length(levels(df1_TotGW$bt))) +
geom_point(aes(shape  = factor(bt), color= factor(ctry))) +
stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
facet_grid(. ~ metier, scales="free") #+
#stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=500,cc=1))
#stat_smooth( method="nls", formula='y~k*sqrt(x)', se=FALSE, start=list(k=50))
update_labels(p, list(x = "kW", y="Total gear weight"))
savePlot(file=file.path(outPath, "plot_OT_TotGW_vs_kW_per_simple_met_ctry.png"), type="png")



##-----------------------------------------
##-----------------------------------------
## Export a table of parameters
## for the Benthis vmstools R workflow
## after having made a choice of the most relevant
## categorisation
##-----------------------------------------
##-----------------------------------------

# get the coeffs for TotGW~ LOA or kW
coeffs_TotGW <- NULL
df1_TotGW$TotGW   <-  an(df1_TotGW$TotGW)
df1_TotGW$LOA     <-  an(df1_TotGW$LOA)
df1_TotGW$kW      <-  an(df1_TotGW$kW)
df1_TotGW_c       <- df1_TotGW[!is.na(df1_TotGW$LOA) & !is.na(df1_TotGW$kW),]  # caution: keep complete cases for a true model comparison...
#for (a_metier in c("OT_CRU", "OT_DMF", "OT_MIX", "OT_MIX_ARA", "OT_MIX_DEM_Benthic", "OT_MIX_DEM_Benthic_Pelagic", "OT_MIX_DPS", "OT_MIX_NEP","OT_MIX_TGS_CTC", "OT_MIX_TGS_OCC", "OT_SPF")){
for (a_metier in c("OT_CRU", "OT_DMF",  "OT_MIX_ARA", "OT_MIX_DEM_Benthic", "OT_MIX_DEM_Benthic_Pelagic", "OT_MIX_DPS", "OT_MIX_NEP","OT_MIX_TGS_CTC",  "OT_SPF")){
a_nls_kW        <- nls(TotGW~a*(kW^b), start=list(a=1,b=1),data=df1_TotGW_c[df1_TotGW_c$metier==a_metier,])
a_lm_kW         <- nls(TotGW~a*kW+b, start=list(a=1,b=1),data=df1_TotGW_c[df1_TotGW_c$metier==a_metier,])
a_nls_LOA       <- nls(TotGW~a*(LOA^b), start=list(a=50,b=1),data=df1_TotGW_c[df1_TotGW_c$metier==a_metier,])
a_lm_LOA        <- nls(TotGW~a*LOA+b, start=list(a=50,b=1),data=df1_TotGW_c[df1_TotGW_c$metier==a_metier,])

#compare goodness of fit
residualSum    <- anova (a_nls_LOA, a_lm_LOA, a_nls_kW, a_lm_kW)
what_is_chosen <- c('a_nls_LOA','a_lm_LOA','a_nls_kW','a_lm_kW') [which.min(residualSum[,2])]
print(what_is_chosen)

nb_records      <- nrow(df1_TotGW[df1_TotGW$metier==a_metier,])

# then choose the best model....
# (and re-run on the full dataset)
if(what_is_chosen=="a_nls_LOA") {
a_nls_LOA      <- nls(TotGW~a*(LOA^b), start=list(a=50,b=1), data=df1_TotGW[df1_TotGW$metier==a_metier,])  # redo with all the available data
coeffs_TotGW     <- rbind.data.frame (coeffs_TotGW,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_nls_LOA)$coeff, equ="TotGW=a*(LOA^b)", nb_records= nb_records))
}
if(what_is_chosen=="a_lm_LOA"){
a_lm_LOA        <- nls(TotGW~a*LOA+b, start=list(a=50,b=1),data=df1_TotGW[df1_TotGW$metier==a_metier,])
coeffs_TotGW      <- rbind.data.frame (coeffs_TotGW,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_lm_LOA)$coeff, equ="TotGW=(a*LOA)+b", nb_records= nb_records))
}
if(what_is_chosen=="a_nls_kW"){
a_nls_kW        <- nls(TotGW~a*(kW^b), start=list(a=1,b=1),data=df1_TotGW[df1_TotGW$metier==a_metier,])
coeffs_TotGW      <- rbind.data.frame (coeffs_TotGW,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_nls_kW)$coeff, equ="TotGW=a*(kW^b)", nb_records= nb_records))
}
if(what_is_chosen=="a_lm_kW"){
a_lm_kW         <- nls(TotGW~a*kW+b, start=list(a=1,b=1),data=df1_TotGW[df1_TotGW$metier==a_metier,])
coeffs_TotGW      <- rbind.data.frame (coeffs_TotGW,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_lm_kW)$coeff, equ="TotGW=(a*kW)+b", nb_records= nb_records))
}
}
rownames(coeffs_TotGW) <- NULL


# export
write.table(coeffs_TotGW, file=file.path(outPath, "estimates_for_OT_nls_TotGW_vs_LOA_or_kW_per_metier.txt"))
# => for using this table in the workflow, partners should link each logbooks records to the metier categories found in that table...



##-------------------------------------------------
##-------------------------------------------------
## do the correponding plot
##-------------------------------------------------
##-------------------------------------------------


#png(filename = file.path(outPath, paste("plot_estimates_for_OT_nls_TotGW_vs_LOA_or_kW_per_metier.png",sep="")),
#                                   width = 1100, height = 2400,
#                                   units = "px", pointsize = 12,  res=300)   # high resolution plot
windows(7,10)
par(mfrow=c(4,3))
par(oma=c(6,4,1,1))
par(mar=c(4,0,2,1))
df1_TotGW$the_colors   <- df1_TotGW$ctry
#library(RColorBrewer)
#the_colors <- brewer.pal(11, "Paired")
#levels(df1_DoS$the_colors) <- the_colors[1:length(unique(df1_DoS$the_colors))]
rgb.palette <- colorRampPalette(c("green", "red", "blue"),
space = "Lab")
levels(df1_TotGW$the_colors) <- rgb.palette (length(levels(df1_TotGW$the_colors)))
df1_TotGW$the_colors <- as.character(df1_TotGW$the_colors)


count <-1
#for(met in c("OT_CRU", "OT_DMF", "OT_MIX", "OT_MIX_ARA", "OT_MIX_DEM_Benthic", "OT_MIX_DEM_Benthic_Pelagic", "OT_MIX_DPS", "OT_MIX_NEP","OT_MIX_TGS_CTC", "OT_MIX_TGS_OCC", "OT_SPF")){
for(met in c("OT_CRU", "OT_DMF",  "OT_MIX_ARA", "OT_MIX_DEM_Benthic", "OT_MIX_DEM_Benthic_Pelagic", "OT_MIX_DPS", "OT_MIX_NEP","OT_MIX_TGS_CTC",  "OT_SPF")){
a           <- coeffs_TotGW[coeffs_TotGW$a_metier==met & coeffs_TotGW$param=='a', 'Estimate']
b           <- coeffs_TotGW[coeffs_TotGW$a_metier==met  & coeffs_TotGW$param=='b', 'Estimate']
an_equation <- as.character(coeffs_TotGW[coeffs_TotGW$a_metier==met,][1,'equ'])
if(length(grep('LOA' , an_equation))>0){
range_LOA <- range(df1_TotGW[df1_TotGW$metier==met, "LOA",], na.rm=TRUE)
LOA       <- seq(range_LOA[1], range_LOA[2], by=1)
plot(df1_TotGW[df1_TotGW$metier==met, "LOA",], df1_TotGW[df1_TotGW$metier==met, "TotGW"], pch=16, col=df1_TotGW[df1_TotGW$metier==met, "the_colors"], xlab="LOA (metre)", ylab="Total gear weight (kg)", axes=FALSE, ylim=c(0,5300))
lines(LOA, eval(parse(text= an_equation)))
}
if(length(grep('kW' , an_equation))>0){
range_kW <- range(df1_TotGW[df1_TotGW$metier==met, "kW",], na.rm=TRUE)
kW       <- seq(range_kW[1], range_kW[2], by=1)
plot(df1_TotGW[df1_TotGW$metier==met, "kW",], df1_TotGW[df1_TotGW$metier==met, "TotGW"], pch=16, col=df1_TotGW[df1_TotGW$metier==met, "the_colors"], xlab="kW", ylab="Total gear weight (kg)", axes=FALSE, ylim=c(0,5300))
lines(kW, eval(parse(text= an_equation)))
}
axis(1)
if(count==1) axis(2, las=2)
box()
title(met, cex=0.7)
count <- count+1
}
plot(0,0,type="n", axes=FALSE, xlab="", ylab="")
legend("topright", legend=unique(df1_TotGW$ctry), fill=unique(df1_TotGW$the_colors), bty="n", cex=1.2, ncol=2)

mtext(side=2, text="Total gear weight (kg)", line=1, outer=TRUE)
savePlot(file=file.path(outPath, "plot_estimates_for_OT_nls_TotGW_vs_LOA_or_kW_per_metier.png"), type="png")

# dev.off()



```



```



# plot (with ggplot2)
library(ggplot2)

# paths
dataPath  <- file.path("C:", "BENTHIS", "data_gear_spec_questionnaire")
outPath   <- file.path("C:", "BENTHIS", "data_gear_spec_questionnaire")



##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
## TBB
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------




# read file
ind_DK  <- read.table(file= file.path(dataPath, 'TBB', 'TBB_DK_28012014.csv'), sep=";", header=TRUE )
ind_NL  <- read.table(file= file.path(dataPath, 'TBB', 'TBB_NL_28012014.csv'), sep=";", header=TRUE )
ind_TUR <- read.table(file= file.path(dataPath, 'TBB', 'TBB_TUR_28012014.csv'), sep=";", header=TRUE )
ind_BEL <- read.table(file= file.path(dataPath, 'TBB', 'TBB_BEL_28012014.csv'), sep=";", header=TRUE )
ind_IT  <- read.table(file= file.path(dataPath, 'TBB', 'TBB_IT_28012014.csv'), sep=";", header=TRUE )

# collate
cols <- c('Anonymous.vessel_ID','Variable.name','Variable','Value')
ind  <- rbind.data.frame (
cbind(country="DK", ind_DK [, cols]),
cbind(country="NL", ind_NL [, cols]),
cbind(country="TUR", ind_TUR [, cols]),
cbind(country="BEL", ind_BEL [, cols]),
cbind(country="IT", ind_IT [, cols])
)


# explore
head(ind [ind$Variable.name=="Trawl_model",])
levels(ind$Variable.name)
an <- function(x) as.numeric(as.character(x))


CT  <- ind [ind$Variable.name=="Consumption_trawling", "Value"]
CS  <- ind [ind$Variable.name=="Consumption_steaming", "Value"]
Str  <- ind [ind$Variable.name=="Speed_trawling", "Value"]
kW  <- ind [ind$Variable.name=="Vessel_kW", "Value"]
LOA <- ind [ind$Variable.name=="Vessel_LOA", "Value"]
beamw <- ind [ind$Variable.name=="Beam width", "Value"]
GrW <- ind [ind$Variable.name=="Groundgear_weight", "Value"]
SuW <- ind [ind$Variable.name=="Sumwingnose_weight", "Value"]
TiW <- ind [ind$Variable.name=="Ticklerchain_weight", "Value"]
TiN <- ind [ind$Variable.name=="Ticklerchain_numbers", "Value"]
GrL <- ind [ind$Variable.name=="Groundgear_length", "Value"]
sps <- ind [ind$Variable.name=="Targetspecies_single", "Value"]
sp1 <- ind [ind$Variable.name=="Primarytarget_mixed", "Value"]
sp2 <- ind [ind$Variable.name=="Secondarytarget_mixed", "Value"]
sp3 <- ind [ind$Variable.name=="Thirdtarget_mixed", "Value"]
bt  <- ind [ind$Variable.name=="Bottom_type", "Value"]
nbt <- ind [ind$Variable.name=="Trawl_number", "Value"]
sptr <- ind [ind$Variable.name=="Speed_trawling", "Value"]
spst <- ind [ind$Variable.name=="Speed_steaming", "Value"]
ctry <- ind [ind$Variable.name=="Trawl_number", "country"]
area <- ind [ind$Variable.name=="Fishing_area", "Value"]
mesh <- ind [ind$Variable.name=="Codend_meshsize", "Value"]

# intermediate calculations
dd    <- rbind(an(GrW) * an(nbt), an(SuW), c(an(TiW) * an(TiN)))
TotGW <- apply(dd,2, sum, na.rm=TRUE) # DoW and GrW need complete cases here.
TotGW[is.na(dd[1,])] <- NA

beamw <- an(nbt) * an(beamw)


# then, collate:
df1  <- cbind.data.frame(ctry,CT, CS, sptr, spst, kW, beamw, LOA, GrW, nbt, TotGW, GrL, sps, sp1, sp2, sp3, bt, area, mesh)




# refactor some variables (if needed)
df1$LOA_class <- cut(an(df1$LOA), breaks=c(0,15,100))
df1$sptr      <- cut(an(df1$sptr), breaks= seq(0,10,0.75))   # trawling speed
df1$mesh      <- cut(an(df1$mesh), breaks= seq(0,140,30))   # codend mesh size


# DCF metier coding
df1$metier <- as.character(df1$sps) # init
df1[df1$metier %in% c('NEP ','NEP','PRA','Nephrops','Nephrops trawl',
'TGS', 'ARA','DPS', 'CRN' ), 'metier'] <- 'TBB_CRU'
df1[df1$metier %in% c('RPW' ), 'metier'] <- 'TBB_MOL'
df1[df1$metier %in% c('COD','PLE','SOL', 'LEM', 'WHG', 'WHI', 'POK',
'PDS','HAD','had','HKE','MON', 'MUT',
'NOP'), 'metier'] <- 'TBB_DMF'
df1[df1$metier %in% c('SAN','SPR','CAP'), 'metier'] <- 'TBB_SPF'
df1[df1$metier %in% c('NR', "NI",'0','','ni'), 'metier'] <- 'TBB_MIX'
df1$metier <- as.factor(df1$metier)




# area coding
df1$area <- as.character(df1$area) # init
df1[df1$area %in% c('IIIa', 'IIIan', 'IIIas', 'IIIast'), 'area'] <- 'kask'
df1[df1$area %in% c('II','IV', 'IVa', 'IVb', 'IVbc','IVc', 'Ivb', 'IV/VII/VIII', 'IV/VII'), 'area'] <- 'nsea'
df1[df1$area %in% c('IIIc', 'IIId', '25'), 'area'] <- 'bsea'
df1[df1$area %in% c('VII','Vb1','VIa', "VIII"), 'area'] <- 'csea'
#df1[df1$area %in% c('2.1', '1.3', '2.2', '1.2','0','1.1',''), 'area'] <- 'msea'
df1[df1$area %in% c('0',''), 'area'] <- 'ni'
df1[df1$area %in% c('Black Sea, Samsun Shelf Area (SSA)'), 'area'] <- 'blsea'
df1$area <- as.factor(df1$area)



# nb of observations per country
table(df1$ctry)

# nb of observations per metier
table(df1$metier)

# nb of observations per country per metier
table(df1$ctry,df1$metier)


##-----------------------------------------
##-----------------------------------------
## trawling vs. steaming consumption in l/h
##-----------------------------------------
##-----------------------------------------

p <- ggplot(data=df1, aes(factor(metier), an(CT)/an(CS))) + geom_boxplot()  + geom_point()
update_labels(p, list(x ="metier" , y="trawling/steaming l/h"))
savePlot(file=file.path(outPath, "plot_ratio_consumption_TBB.png"), type="png")




##-------------------------------------------------
##-------------------------------------------------
## beam width vs. kW
##-------------------------------------------------
##-------------------------------------------------

# look at the representativity....
df1$informedbeamw <- ifelse(is.na(an(df1$beamw)) | an(df1$beamw)==0,0,1)
table(df1$ctry, df1$informedbeamw)

# look at the representativity....
df1$informedarea <- ifelse(as.character(df1$area)=='0',0,1)
table(df1$ctry, df1$informedarea)


# look at the representativity....
table(df1$ctry, df1$metier, df1$informedbeamw)

# filter out the 0
df1_beamw <- df1[!is.na(df1$beamw) & df1$beamw!=0 & an(df1$beamw)<1000,]

# plot beamw
library(ggplot2)
coeff_lm <- coef(lm(an(beamw) ~ an(kW) , data = df1_beamw))
p <- ggplot(data=df1_beamw, aes(x=an(kW),y=an(beamw), color= factor(metier))) +
geom_point(aes(shape  = factor(metier), color= factor(metier))) +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=1,cc=1))
#stat_smooth( method="glm", family="poisson", formula='DoS~kW+metier')
# stat_smooth(method="lm")
update_labels(p, list(x = "kW", y="Beam_width"))
savePlot(file=file.path(outPath, "plot_TBB_nls_beamw_vs_kW_per_metier.png"), type="png")


# plot beamw  nls fit with LOA and ctry
library(ggplot2)
coeff_lm <- coef(lm(an(beamw) ~ an(kW) , data = df1_beamw))
p <- ggplot(data=df1_beamw, aes(x=an(kW),y=an(beamw))) +
geom_point(aes(shape  = factor(LOA_class), color= factor(ctry))) +
#stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
facet_grid(. ~ metier, scales="free") +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=50,cc=1))
update_labels(p, list(x = "kW", y="Beam_width"))
savePlot(file=file.path(outPath, "plot_TBB_nls_beamw_vs_kW_per_LOA_ctry.png"), type="png")


##-----------------------------------------
##-----------------------------------------
## Export a table of parameters
## for the Benthis vmstools R workflow
## after having made a choice of the most relevant
## categorisation
##-----------------------------------------
##-----------------------------------------

# get the coeffs for TotGW~ LOA or kW
coeffs_beamw <- NULL
df1_beamw$beamw   <-  an(df1_beamw$beamw)
df1_beamw$LOA     <-  an(df1_beamw$LOA)
df1_beamw$kW      <-  an(df1_beamw$kW)
df1_beamw_c       <- df1_beamw[!is.na(df1_beamw$LOA) & !is.na(df1_beamw$kW),]  # caution: keep complete cases for a true model comparison...
for (a_metier in c("TBB_CRU", "TBB_DMF", "TBB_MIX", "TBB_MOL")){
a_nls_kW        <- nls(beamw~a*(kW^b), start=list(a=1,b=1),data=df1_beamw_c[df1_beamw_c$metier==a_metier,])
a_lm_kW         <- nls(beamw~a*kW+b, start=list(a=1,b=1),data=df1_beamw_c[df1_beamw_c$metier==a_metier,])
a_nls_LOA       <- nls(beamw~a*(LOA^b), start=list(a=1,b=1),data=df1_beamw_c[df1_beamw_c$metier==a_metier,])
a_lm_LOA        <- nls(beamw~a*LOA+b, start=list(a=1,b=1),data=df1_beamw_c[df1_beamw_c$metier==a_metier,])

#compare goodness of fit
residualSum    <- anova (a_nls_LOA, a_lm_LOA, a_nls_kW, a_lm_kW)
what_is_chosen <- c('a_nls_LOA','a_lm_LOA','a_nls_kW','a_lm_kW') [which.min(residualSum[,2])]
print(what_is_chosen)

nb_records      <- nrow(df1_beamw[df1_beamw$metier==a_metier,])

# then choose the best model....
# (and re-run on the full dataset)
if(what_is_chosen=="a_nls_LOA") {
a_nls_LOA      <- nls(beamw~a*(LOA^b), start=list(a=1,b=1), data=df1_beamw[df1_beamw$metier==a_metier,])  # redo with all the available data
coeffs_beamw     <- rbind.data.frame (coeffs_beamw,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_nls_LOA)$coeff, equ="beamw=a*(LOA^b)", nb_records= nb_records))
}
if(what_is_chosen=="a_lm_LOA"){
a_lm_LOA        <- nls(beamw~a*LOA+b, start=list(a=1,b=1),data=df1_beamw[df1_beamw$metier==a_metier,])
coeffs_beamw      <- rbind.data.frame (coeffs_beamw,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_lm_LOA)$coeff, equ="beamw=(a*LOA)+b", nb_records= nb_records))
}
if(what_is_chosen=="a_nls_kW"){
a_nls_kW        <- nls(beamw~a*(kW^b), start=list(a=1,b=1),data=df1_beamw[df1_beamw$metier==a_metier,])
coeffs_beamw      <- rbind.data.frame (coeffs_beamw,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_nls_kW)$coeff, equ="beamw=a*(kW^b)", nb_records= nb_records))
}
if(what_is_chosen=="a_lm_kW"){
a_lm_kW         <- nls(beamw~a*kW+b, start=list(a=1,b=1),data=df1_beamw[df1_beamw$metier==a_metier,])
coeffs_beamw      <- rbind.data.frame (coeffs_beamw,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_lm_kW)$coeff, equ="beamw=(a*kW)+b", nb_records= nb_records))
}
}
rownames(coeffs_beamw) <- NULL


# export
write.table(coeffs_beamw, file=file.path(outPath, "estimates_for_TBB_nls_beamw_vs_LOA_or_kW_per_metier.txt"))
# => for using this table in the workflow, partners should link each logbooks records to the metier categories found in that table...



##-------------------------------------------------
##-------------------------------------------------
## do the correponding plot
##-------------------------------------------------
##-------------------------------------------------


#png(filename = file.path(outPath, paste("plot_estimates_for_TBB_nls_TotGW_vs_LOA_or_kW_per_metier.png",sep="")),
#                                   width = 1100, height = 2400,
#                                   units = "px", pointsize = 12,  res=300)   # high resolution plot
windows(10,5)
par(mfrow=c(1,5))
par(oma=c(6,4,1,1))
par(mar=c(4,0,2,1))
df1_beamw$the_colors   <- df1_beamw$ctry
#library(RColorBrewer)
#the_colors <- brewer.pal(11, "Paired")
#levels(df1_beamw$the_colors) <- the_colors[1:length(unique(df1_beamw$the_colors))]
rgb.palette <- colorRampPalette(c("green", "red", "blue"),
space = "Lab")
levels(df1_beamw$the_colors) <- rgb.palette (length(levels(df1_beamw$the_colors)))
df1_beamw$the_colors <- as.character(df1_beamw$the_colors)


count <-1
for (met in c("TBB_CRU", "TBB_DMF", "TBB_MIX", "TBB_MOL")){
a           <- coeffs_beamw[coeffs_beamw$a_metier==met & coeffs_beamw$param=='a', 'Estimate']
b           <- coeffs_beamw[coeffs_beamw$a_metier==met  & coeffs_beamw$param=='b', 'Estimate']
an_equation <- as.character(coeffs_beamw[coeffs_beamw$a_metier==met,][1,'equ'])
if(length(grep('LOA' , an_equation))>0){
range_LOA <- range(df1_beamw[df1_beamw$metier==met, "LOA",], na.rm=TRUE)
LOA       <- seq(range_LOA[1], range_LOA[2], by=1)
plot(df1_beamw[df1_beamw$metier==met, "LOA",], df1_beamw[df1_beamw$metier==met, "beamw"], pch=16, col=df1_beamw[df1_beamw$metier==met, "the_colors"], xlab="LOA (metre)", ylab="Total gear weight (kg)", axes=FALSE, ylim=c(0,30))
lines(LOA, eval(parse(text= an_equation)))
}
if(length(grep('kW' , an_equation))>0){
range_kW <- range(df1_beamw[df1_beamw$metier==met, "kW",], na.rm=TRUE)
kW       <- seq(range_kW[1], range_kW[2], by=1)
plot(df1_beamw[df1_beamw$metier==met, "kW",], df1_beamw[df1_beamw$metier==met, "beamw"], pch=16, col=df1_beamw[df1_beamw$metier==met, "the_colors"], xlab="kW", ylab="Total gear weight (kg)", axes=FALSE, ylim=c(0,30))
lines(kW, eval(parse(text= an_equation)))
}
axis(1)
if(count==1) axis(2, las=2)
box()
title(met, cex=0.7)
count <- count+1
}
plot(0,0,type="n", axes=FALSE, xlab="", ylab="")
legend("topright", legend=unique(df1_beamw$ctry), fill=unique(df1_beamw$the_colors), bty="n", cex=1.2, ncol=2)

mtext(side=2, text="Beam width * nb. of trawls (metre)", line=2.5, outer=TRUE)
savePlot(file=file.path(outPath, "plot_estimates_for_TBB_nls_beamw_vs_LOA_or_kW_per_metier.png"), type="png")

# dev.off()







##-------------------------------------------------
##-------------------------------------------------
## total gear weight vs. kW
##-------------------------------------------------
##-------------------------------------------------


# look at the representativity....
df1$informedTotGW<- ifelse(is.na(an(df1$TotGW)) | an(df1$TotGW)==0,0,1)
table(df1$ctry, df1$informedTotGW)

# look at the representativity....
df1$informedarea <- ifelse(as.character(df1$area)=='0',0,1)
table(df1$ctry, df1$informedarea)



# filter out the 0
df1_TotGW <- df1[!is.na(df1$TotGW) & df1$TotGW!=0,]

# plot beamw
library(ggplot2)
coeff_lm <- coef(lm(an(TotGW) ~ an(kW) , data = df1_TotGW))
p <- ggplot(data=df1_TotGW, aes(x=an(kW),y=an(TotGW), color= factor(metier))) +
geom_point(aes(shape  = factor(metier), color= factor(metier))) +
stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
#stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=1,cc=1))
#stat_smooth( method="glm", family="poisson", formula='DoS~kW+metier')
# stat_smooth(method="lm")
facet_grid(. ~ metier, scales="free")

update_labels(p, list(x = "kW", y="TotGW"))
savePlot(file=file.path(outPath, "plot_TBB_TotGW_vs_kW_per_metier.png"), type="png")







```

For the DRB gear:

```




# plot (with ggplot2)
library(ggplot2)

# paths
dataPath  <- file.path("C:", "BENTHIS", "data_gear_spec_questionnaire")
outPath   <- file.path("C:", "BENTHIS", "data_gear_spec_questionnaire")



##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
## DRB
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------




# read file
ind_IRE <- read.table(file= file.path(dataPath, 'DRB', 'DRB_IRE_28012014.csv'), sep=";", header=TRUE )

# collate
cols <- c('Anonymous.vessel_ID','Variable.name','Variable','Value')
ind  <- rbind.data.frame (
cbind(country="IRE", ind_IRE [, cols])
)


# explore
head(ind [ind$Variable.name=="Dredge_model",])
levels(ind$Variable.name)
an <- function(x) as.numeric(as.character(x))


CT  <- ind [ind$Variable.name=="Consumption_dredging", "Value"]
CS  <- ind [ind$Variable.name=="Consumption_steaming", "Value"]
kW  <- ind [ind$Variable.name=="Vessel_kW", "Value"]
LOA <- ind [ind$Variable.name=="Vessel_LOA", "Value"]
dredgew <- ind [ind$Variable.name=="Dredge_width", "Value"]
GrW <- ind [ind$Variable.name=="Dredge_weight", "Value"]
sps <- ind [ind$Variable.name=="Targetspecies_single", "Value"]
sp1 <- ind [ind$Variable.name=="Primarytarget_mixed", "Value"]
sp2 <- ind [ind$Variable.name=="Secondarytarget_mixed", "Value"]
sp3 <- ind [ind$Variable.name=="Thirdtarget_mixed", "Value"]
bt  <- ind [ind$Variable.name=="Bottom_type", "Value"]
nbt <- ind [ind$Variable.name=="Dredge_number", "Value"]
sptr <- ind [ind$Variable.name=="Speed_dredging", "Value"]
spst <- ind [ind$Variable.name=="Speed_steaming", "Value"]
ctry <- ind [ind$Variable.name=="Dredge_number", "country"]
area <- ind [ind$Variable.name=="Fishing_area", "Value"]

# intermediate calculations
TotGW   <- an(GrW) * an(nbt)
TotGW[is.na(TotGW[1])] <- NA
dredgew <- an(nbt) * an(dredgew)


# then, collate:
df1  <- cbind.data.frame(ctry,CT, CS, sptr, spst, kW, dredgew, LOA, GrW, nbt, TotGW, sps, sp1, sp2, sp3, bt, area)




# refactor some variables (if needed)
df1$LOA_class <- cut(an(df1$LOA), breaks=c(0,15,100))
df1$sptr      <- cut(an(df1$sptr), breaks= seq(0,10,0.75))   # trawling speed


# DCF metier coding
df1$metier <- as.character(df1$sps) # init
df1[df1$metier %in% c('SCE'), 'metier'] <- 'DRB_MOL'
df1$metier <- as.factor(df1$metier)




# area coding
df1$area <- as.character(df1$area) # init
df1$area <- as.factor(df1$area)



# nb of observations per country
table(df1$ctry)

# nb of observations per metier
table(df1$metier)

# nb of observations per country per metier
table(df1$ctry,df1$metier)



##-------------------------------------------------
##-------------------------------------------------
## dredge width vs. kW
##-------------------------------------------------
##-------------------------------------------------

# look at the representativity....
df1$informeddredgew <- ifelse(is.na(an(df1$dredgew)) | an(df1$dredgew)==0,0,1)
table(df1$ctry, df1$informeddredgew)

# look at the representativity....
df1$informedarea <- ifelse(as.character(df1$area)=='0',0,1)
table(df1$ctry, df1$informedarea)


# look at the representativity....
table(df1$ctry, df1$metier, df1$informeddredgew)

# filter out the 0
df1_dredgew <- df1[!is.na(df1$dredgew) & df1$dredgew!=0,]

# plot dredgew
library(ggplot2)
coeff_lm <- coef(lm(an(dredgew) ~ an(kW) , data = df1_dredgew))
p <- ggplot(data=df1_dredgew, aes(x=an(kW),y=an(dredgew), color= factor(metier))) +
geom_point(aes(shape  = factor(metier), color= factor(metier))) +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=1,cc=1))
#stat_smooth( method="glm", family="poisson", formula='DoS~kW+metier')
# stat_smooth(method="lm")
update_labels(p, list(x = "kW", y="Dredge_width"))
savePlot(file=file.path(outPath, "plot_TBB_nls_dredgew_vs_kW_per_metier.png"), type="png")


# plot dredgew  nls fit with LOA and ctry
library(ggplot2)
coeff_lm <- coef(lm(an(dredgew) ~ an(kW) , data = df1_dredgew))
p <- ggplot(data=df1_dredgew, aes(x=an(kW),y=an(dredgew))) +
geom_point(aes(shape  = factor(LOA_class), color= factor(ctry))) +
#stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
facet_grid(. ~ metier, scales="free") +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=50,cc=1))
update_labels(p, list(x = "kW", y="Dredge_width"))
savePlot(file=file.path(outPath, "plot_TBB_nls_dredgew_vs_kW_per_LOA_ctry.png"), type="png")


##-----------------------------------------
##-----------------------------------------
## Export a table of parameters
## for the Benthis vmstools R workflow
## after having made a choice of the most relevant
## categorisation
##-----------------------------------------
##-----------------------------------------

# get the coeffs for TotGW~ LOA or kW
coeffs_dredgew <- NULL
df1_dredgew$dredgew   <-  an(df1_dredgewdredgew)
df1_dredgew$LOA     <-  an(df1_dredgew$LOA)
df1_dredgew$kW      <-  an(df1_dredgew$kW)
df1_dredgew_c       <- df1_dredgew[!is.na(df1_dredgew$LOA) & !is.na(df1_dredgew$kW),]  # caution: keep complete cases for a true model comparison...
for (a_metier in c("DRB_MOL")){
a_nls_kW        <- nls(dredgew~a*(kW^b), start=list(a=1,b=1),data=df1_dredgew_c[df1_dredgew_c$metier==a_metier,])
a_lm_kW         <- nls(dredgew~a*kW+b, start=list(a=1,b=1),data=df1_dredgew_c[df1_dredgew_c$metier==a_metier,])
a_nls_LOA       <- nls(dredgew~a*(LOA^b), start=list(a=1,b=1),data=df1_dredgew_c[df1_dredgew_c$metier==a_metier,])
a_lm_LOA        <- nls(dredgew~a*LOA+b, start=list(a=1,b=1),data=df1_dredgew_c[df1_dredgew_c$metier==a_metier,])

#compare goodness of fit
residualSum    <- anova (a_nls_LOA, a_lm_LOA, a_nls_kW, a_lm_kW)
what_is_chosen <- c('a_nls_LOA','a_lm_LOA','a_nls_kW','a_lm_kW') [which.min(residualSum[,2])]
print(what_is_chosen)

nb_records      <- nrow(df1_dredgew[df1_dredgew$metier==a_metier,])

# then choose the best model....
# (and re-run on the full dataset)
if(what_is_chosen=="a_nls_LOA") {
a_nls_LOA      <- nls(dredgew~a*(LOA^b), start=list(a=1,b=1), data=df1_dredgew[df1_dredgew$metier==a_metier,])  # redo with all the available data
coeffs_dredgew     <- rbind.data.frame (coeffs_dredgew,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_nls_LOA)$coeff, equ="dredgew=a*(LOA^b)", nb_records= nb_records))
}
if(what_is_chosen=="a_lm_LOA"){
a_lm_LOA        <- nls(dredgew~a*LOA+b, start=list(a=1,b=1),data=df1_dredgew[df1_dredgew$metier==a_metier,])
coeffs_dredgew      <- rbind.data.frame (coeffs_dredgew,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_lm_LOA)$coeff, equ="dredgew=(a*LOA)+b", nb_records= nb_records))
}
if(what_is_chosen=="a_nls_kW"){
a_nls_kW        <- nls(dredgew~a*(kW^b), start=list(a=1,b=1),data=df1_dredgew[df1_dredgew$metier==a_metier,])
coeffs_dredgew      <- rbind.data.frame (coeffs_dredgew,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_nls_kW)$coeff, equ="dredgew=a*(kW^b)", nb_records= nb_records))
}
if(what_is_chosen=="a_lm_kW"){
a_lm_kW         <- nls(dredgew~a*kW+b, start=list(a=1,b=1),data=df1_dredgew[df1_dredgew$metier==a_metier,])
coeffs_dredgew      <- rbind.data.frame (coeffs_dredgew,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_lm_kW)$coeff, equ="dredgew=(a*kW)+b", nb_records= nb_records))
}
}
rownames(coeffs_dredgew) <- NULL


# export
write.table(coeffs_dredgew, file=file.path(outPath, "estimates_for_DRB_nls_dredgew_vs_LOA_or_kW_per_metier.txt"))
# => for using this table in the workflow, partners should link each logbooks records to the metier categories found in that table...



##-------------------------------------------------
##-------------------------------------------------
## do the correponding plot
##-------------------------------------------------
##-------------------------------------------------


#png(filename = file.path(outPath, paste("plot_estimates_for_DRB_nls_TotGW_vs_LOA_or_kW_per_metier.png",sep="")),
#                                   width = 1100, height = 2400,
#                                   units = "px", pointsize = 12,  res=300)   # high resolution plot
windows(10,5)
par(mfrow=c(1,2))
par(oma=c(6,4,1,1))
par(mar=c(4,0,2,1))
df1_dredgew$the_colors   <- df1_dredgew$ctry
#library(RColorBrewer)
#the_colors <- brewer.pal(11, "Paired")
#levels(df1_dredgew$the_colors) <- the_colors[1:length(unique(df1_dredgew$the_colors))]
rgb.palette <- colorRampPalette(c("green", "red", "blue"),
space = "Lab")
levels(df1_dredgew$the_colors) <- rgb.palette (length(levels(df1_dredgew$the_colors)))
df1_dredgew$the_colors <- as.character(df1_dredgew$the_colors)


count <-1
for (met in c("DRB_MOL")){
a           <- coeffs_dredgew[coeffs_dredgew$a_metier==met & coeffs_dredgew$param=='a', 'Estimate']
b           <- coeffs_dredgew[coeffs_dredgew$a_metier==met  & coeffs_dredgew$param=='b', 'Estimate']
an_equation <- as.character(coeffs_dredgew[coeffs_dredgew$a_metier==met,][1,'equ'])
if(length(grep('LOA' , an_equation))>0){
range_LOA <- range(df1_dredgew[df1_dredgew$metier==met, "LOA",], na.rm=TRUE)
LOA       <- seq(range_LOA[1], range_LOA[2], by=1)
plot(df1_dredgew[df1_dredgew$metier==met, "LOA",], df1_dredgew[df1_dredgew$metier==met, "dredgew"], pch=16, col=df1_dredgew[df1_dredgew$metier==met, "the_colors"], xlab="LOA (metre)", ylab="Total gear weight (kg)", axes=FALSE, ylim=c(0,30))
lines(LOA, eval(parse(text= an_equation)))
}
if(length(grep('kW' , an_equation))>0){
range_kW <- range(df1_dredgew[df1_dredgew$metier==met, "kW",], na.rm=TRUE)
kW       <- seq(range_kW[1], range_kW[2], by=1)
plot(df1_dredgew[df1_dredgew$metier==met, "kW",], df1_dredgew[df1_dredgew$metier==met, "dredgew"], pch=16, col=df1_dredgew[df1_dredgew$metier==met, "the_colors"], xlab="kW", ylab="Total gear weight (kg)", axes=FALSE, ylim=c(0,30))
lines(kW, eval(parse(text= an_equation)))
}
axis(1)
if(count==1) axis(2, las=2)
box()
title(met, cex=0.7)
count <- count+1
}
plot(0,0,type="n", axes=FALSE, xlab="", ylab="")
legend("topright", legend=unique(df1_dredgew$ctry), fill=unique(df1_dredgew$the_colors), bty="n", cex=1.2, ncol=2)

mtext(side=2, text="Dredge width * nb. of trawls (metre)", line=2.5, outer=TRUE)
savePlot(file=file.path(outPath, "plot_estimates_for_DRB_nls_dredgew_vs_LOA_or_kW_per_metier.png"), type="png")

# dev.off()







##-------------------------------------------------
##-------------------------------------------------
## total gear weight vs. kW
##-------------------------------------------------
##-------------------------------------------------


# look at the representativity....
df1$informedTotGW<- ifelse(is.na(an(df1$TotGW)) | an(df1$TotGW)==0,0,1)
table(df1$ctry, df1$informedTotGW)

#=> GEAR WEIGHT NOT INFORMED! USE LITTERATURE INSTEAD.............




```

For the demersal seine gear, the length of the rope is related either to vessel kW or LOA. The calculation of the swept area of a given fishing event require further assumptions related to the shape of the fishing gear deployment and the duration of the fishing event. These assumptions are incorporated within the BENTHIS WP2 workflow when it comes to compute the swept area from the length of the rope

(in brief it has been assumed that the average duration of a fishing event is 3 hours and the shape is a circle area centered on the fishing ping, no difference between Danish and Scottish seiners which is an approximation).

```




# plot (with ggplot2)
library(ggplot2)

# paths
dataPath  <- file.path("C:", "BENTHIS", "data_gear_spec_questionnaire")
outPath   <- file.path("C:", "BENTHIS", "data_gear_spec_questionnaire")



##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
## DS
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------
##-------------------------------------------------




# read file
ind_DK  <- read.table(file= file.path(dataPath, 'DS', 'DS_DK_04022014.csv'), sep=";", header=TRUE )
ind_NOR  <- read.table(file= file.path(dataPath, 'DS', 'DS_NOR_29012014.csv'), sep=";", header=TRUE )

# collate
cols <- c('Anonymous.vessel_ID','Variable.name','Variable','Value')
ind  <- rbind.data.frame (
cbind(country="DK", ind_DK [, cols]),
cbind(country="NOR", ind_NOR [, cols])
)


# explore
head(ind [ind$Variable.name=="Seine_model",])
levels(ind$Variable.name)
an <- function(x) as.numeric(as.character(x))


CT  <- ind [ind$Variable.name=="Consumption_fishing", "Value"]
CS  <- ind [ind$Variable.name=="Consumption_steaming", "Value"]
kW  <- ind [ind$Variable.name=="Vessel_kW", "Value"]
LOA <- ind [ind$Variable.name=="Vessel_LOA", "Value"]
seineropel <- ind [ind$Variable.name=="Seinerope_length", "Value"]
GrW <- ind [ind$Variable.name=="Groundgear_weight", "Value"]
GrL <- ind [ind$Variable.name=="Groundgear_length", "Value"]
sps <- ind [ind$Variable.name=="Targetspecies_single", "Value"]
sp1 <- ind [ind$Variable.name=="Primarytarget_mixed", "Value"]
sp2 <- ind [ind$Variable.name=="Secondarytarget_mixed", "Value"]
sp3 <- ind [ind$Variable.name=="Thirdtarget_mixed", "Value"]
bt  <- ind [ind$Variable.name=="Bottom_type", "Value"]
spst <- ind [ind$Variable.name=="Speed_steaming", "Value"]
area <- ind [ind$Variable.name=="Fishing_area", "Value"]
ctry <- ind [ind$Variable.name=="Seinerope_length", "country"]
mesh <- ind [ind$Variable.name=="Codend_meshsize", "Value"]
gear <-  ind [ind$Variable.name=="Seine_model", "Value"]

# then, collate:
df1  <- cbind.data.frame(ctry,CT, CS, spst, kW, seineropel, LOA, GrW, GrL, sps, sp1, sp2, sp3, area, mesh, gear)




# refactor some variables (if needed)
df1$LOA_class <- cut(an(df1$LOA), breaks=c(0,15,100))
df1$mesh      <- cut(an(df1$mesh), breaks= seq(0,140,30))   # codend mesh size


# DCF metier coding
df1$metier <- as.character(df1$gear) # init
df1[df1$metier %in% c('Danish'), 'metier'] <- 'SDN_DEM'
df1[df1$metier %in% c('Scottish'), 'metier'] <- 'SSC_DEM'
df1$metier <- as.factor(df1$metier)




# area coding
df1$area <- as.character(df1$area) # init
df1[df1$area %in% c('IIIa', 'IIIan', 'IIIas', 'IIIast'), 'area'] <- 'kask'
df1[df1$area %in% c('II','IV', 'IVa', 'IVb', 'IVbc','IVc', 'Ivb', 'IV/VII/VIII', 'IV/VII'), 'area'] <- 'nsea'
df1[df1$area %in% c('IIIc', 'IIId', '25'), 'area'] <- 'bsea'
df1[df1$area %in% c('VII','Vb1','VIa', "VIII"), 'area'] <- 'csea'
#df1[df1$area %in% c('2.1', '1.3', '2.2', '1.2','0','1.1',''), 'area'] <- 'msea'
df1[df1$area %in% c('0',''), 'area'] <- 'ni'
df1[df1$area %in% c('Black Sea, Samsun Shelf Area (SSA)'), 'area'] <- 'blsea'
df1[df1$area %in% c('Barentshave'), 'area'] <- 'barsea'
df1$area <- as.factor(df1$area)



# nb of observations per country
table(df1$ctry)

# nb of observations per metier
table(df1$metier)

# nb of observations per country per metier
table(df1$ctry,df1$metier)




##-------------------------------------------------
##-------------------------------------------------
## beam width vs. kW
##-------------------------------------------------
##-------------------------------------------------

# look at the representativity....
df1$informedseineropel <- ifelse(is.na(an(df1$seineropel)) | an(df1$seineropel)==0,0,1)
table(df1$ctry, df1$informedseineropel)

# look at the representativity....
df1$informedarea <- ifelse(as.character(df1$area)=='0',0,1)
table(df1$ctry, df1$informedarea)


# look at the representativity....
table(df1$ctry, df1$metier, df1$informedseineropel)

# filter out the 0
df1_seineropel <- df1[!is.na(df1$seineropel) & df1$seineropel!=0,]


# plot seineropel  nls fit with LOA and ctry
library(ggplot2)
coeff_lm <- coef(lm(an(seineropel) ~ an(kW) , data = df1_seineropel))
p <- ggplot(data=df1_seineropel, aes(x=an(kW),y=an(seineropel))) +
geom_point(aes(shape  = factor(LOA_class), color= factor(ctry))) +
#stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
facet_grid(. ~ metier, scales="free") +
stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=50,cc=1))
update_labels(p, list(x = "kW", y="Seine rope length"))
savePlot(file=file.path(outPath, "plot_DS_nls_seineropel_vs_kW_per_LOA_ctry.png"), type="png")


##-----------------------------------------
##-----------------------------------------
## Export a table of parameters
## for the Benthis vmstools R workflow
## after having made a choice of the most relevant
## categorisation
##-----------------------------------------
##-----------------------------------------

# get the coeffs for GrW~ LOA or kW
coeffs_seineropel <- NULL
df1_seineropel$seineropel   <-  an(df1_seineropel$seineropel)
df1_seineropel$LOA     <-  an(df1_seineropel$LOA)
df1_seineropel$kW      <-  an(df1_seineropel$kW)
df1_seineropel_c       <- df1_seineropel[!is.na(df1_seineropel$LOA) & !is.na(df1_seineropel$kW),]  # caution: keep complete cases for a true model comparison...
for (a_metier in c("SDN_DEM", "SSC_DEM")){
a_nls_kW        <- nls(seineropel~a*(kW^b), start=list(a=100,b=1),data=df1_seineropel_c[df1_seineropel_c$metier==a_metier,])
a_lm_kW         <- nls(seineropel~a*kW+b, start=list(a=100,b=1),data=df1_seineropel_c[df1_seineropel_c$metier==a_metier,])
a_nls_LOA       <- nls(seineropel~a*(LOA^b), start=list(a=100,b=1),data=df1_seineropel_c[df1_seineropel_c$metier==a_metier,])
a_lm_LOA        <- nls(seineropel~a*LOA+b, start=list(a=100,b=1),data=df1_seineropel_c[df1_seineropel_c$metier==a_metier,])

#compare goodness of fit
residualSum    <- anova (a_nls_LOA, a_lm_LOA, a_nls_kW, a_lm_kW)
what_is_chosen <- c('a_nls_LOA','a_lm_LOA','a_nls_kW','a_lm_kW') [which.min(residualSum[,2])]
print(what_is_chosen)

nb_records      <- nrow(df1_seineropel[df1_seineropel$metier==a_metier,])

# then choose the best model....
# (and re-run on the full dataset)
if(what_is_chosen=="a_nls_LOA") {
a_nls_LOA      <- nls(seineropel~a*(LOA^b), start=list(a=100,b=1), data=df1_seineropel[df1_seineropel$metier==a_metier,])  # redo with all the available data
coeffs_seineropel     <- rbind.data.frame (coeffs_seineropel,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_nls_LOA)$coeff, equ="seineropel=a*(LOA^b)", nb_records= nb_records))
}
if(what_is_chosen=="a_lm_LOA"){
a_lm_LOA        <- nls(seineropel~a*LOA+b, start=list(a=100,b=1),data=df1_seineropel[df1_seineropel$metier==a_metier,])
coeffs_seineropel      <- rbind.data.frame (coeffs_seineropel,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_lm_LOA)$coeff, equ="seineropel=(a*LOA)+b", nb_records= nb_records))
}
if(what_is_chosen=="a_nls_kW"){
a_nls_kW        <- nls(seineropel~a*(kW^b), start=list(a=1,b=1),data=df1_seineropel[df1_seineropel$metier==a_metier,])
coeffs_seineropel      <- rbind.data.frame (coeffs_seineropel,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_nls_kW)$coeff, equ="seineropel=a*(kW^b)", nb_records= nb_records))
}
if(what_is_chosen=="a_lm_kW"){
a_lm_kW         <- nls(seineropel~a*kW+b, start=list(a=1,b=1),data=df1_seineropel[df1_seineropel$metier==a_metier,])
coeffs_seineropel      <- rbind.data.frame (coeffs_seineropel,  cbind.data.frame(a_metier, param=c('a','b'), summary(a_lm_kW)$coeff, equ="seineropel=(a*kW)+b", nb_records= nb_records))
}
}
rownames(coeffs_seineropel) <- NULL


# export
write.table(coeffs_seineropel, file=file.path(outPath, "estimates_for_DS_nls_seineropel_vs_LOA_or_kW_per_metier.txt"))
# => for using this table in the workflow, partners should link each logbooks records to the metier categories found in that table...



##-------------------------------------------------
##-------------------------------------------------
## do the correponding plot
##-------------------------------------------------
##-------------------------------------------------


#png(filename = file.path(outPath, paste("plot_estimates_for_DS_nls_GrW_vs_LOA_or_kW_per_metier.png",sep="")),
#                                   width = 1100, height = 2400,
#                                   units = "px", pointsize = 12,  res=300)   # high resolution plot
windows(10,5)
par(mfrow=c(1,3))
par(oma=c(6,4,1,1))
par(mar=c(4,2.8,2,1))
df1_seineropel$the_colors   <- df1_seineropel$ctry
#library(RColorBrewer)
#the_colors <- brewer.pal(11, "Paired")
#levels(df1_seineropel$the_colors) <- the_colors[1:length(unique(df1_seineropel$the_colors))]
rgb.palette <- colorRampPalette(c("green", "red", "blue"),
space = "Lab")
levels(df1_seineropel$the_colors) <- rgb.palette (length(levels(df1_seineropel$the_colors)))
df1_seineropel$the_colors <- as.character(df1_seineropel$the_colors)


count <-1
for (a_metier in c("SDN_DEM", "SSC_DEM")){
a           <- coeffs_seineropel[coeffs_seineropel$a_metier==a_metier & coeffs_seineropel$param=='a', 'Estimate']
b           <- coeffs_seineropel[coeffs_seineropel$a_metier==a_metier  & coeffs_seineropel$param=='b', 'Estimate']
an_equation <- as.character(coeffs_seineropel[coeffs_seineropel$a_metier==a_metier,][1,'equ'])
if(length(grep('LOA' , an_equation))>0){
range_LOA <- range(df1_seineropel[df1_seineropel$metier==a_metier, "LOA",], na.rm=TRUE)
LOA       <- seq(range_LOA[1], range_LOA[2], by=1)
plot(df1_seineropel[df1_seineropel$metier==a_metier, "LOA",], df1_seineropel[df1_seineropel$metier==a_metier, "seineropel"], pch=16, col=df1_seineropel[df1_seineropel$metier==a_metier, "the_colors"], xlab="LOA (metre)", ylab="Seine rope length (m)", axes=FALSE, ylim=c(0,7000))
lines(LOA, eval(parse(text= an_equation)))
}
if(length(grep('kW' , an_equation))>0){
range_kW <- range(df1_seineropel[df1_seineropel$metier==a_metier, "kW",], na.rm=TRUE)
kW       <- seq(range_kW[1], range_kW[2], by=1)
plot(df1_seineropel[df1_seineropel$metier==a_metier, "kW",], df1_seineropel[df1_seineropel$metier==a_metier, "seineropel"], pch=16, col=df1_seineropel[df1_seineropel$metier==a_metier, "the_colors"], xlab="kW", ylab="Seine rope length (m)", axes=FALSE, ylim=c(0,7000))
lines(kW, eval(parse(text= an_equation)))
}
axis(1)
if(count==1) axis(2, las=2)
box()
title(a_metier, cex=0.7)
count <- count+1
}
plot(0,0,type="n", axes=FALSE, xlab="", ylab="")
legend("topright", legend=unique(df1_seineropel$ctry), fill=unique(df1_seineropel$the_colors), bty="n", cex=1.2, ncol=2)

mtext(side=2, text="Seine rope length (metre)", line=2.5, outer=TRUE)
savePlot(file=file.path(outPath, "plot_estimates_for_DS_nls_seineropel_vs_LOA_or_kW_per_metier.png"), type="png")

# dev.off()







##-------------------------------------------------
##-------------------------------------------------
## total gear weight vs. kW
##-------------------------------------------------
##-------------------------------------------------


# look at the representativity....
df1$informedGrW<- ifelse(is.na(an(df1$GrW)) | an(df1$GrW)==0,0,1)
table(df1$ctry, df1$informedGrW)

# look at the representativity....
df1$informedarea <- ifelse(as.character(df1$area)=='0',0,1)
table(df1$ctry, df1$informedarea)



# filter out the 0
df1_GrW <- df1[!is.na(df1$GrW) & df1$GrW!=0,]

# plot seineropel
library(ggplot2)
coeff_lm <- coef(lm(an(GrW) ~ an(kW) , data = df1_GrW))
p <- ggplot(data=df1_GrW, aes(x=an(kW),y=an(GrW), color= factor(metier))) +
geom_point(aes(shape  = factor(metier), color= factor(metier))) +
stat_smooth(method="lm") +
#geom_abline(intercept = coeff_lm[1], slope = coeff_lm[2]) +
#stat_smooth( method="nls", formula='y~b*(x^cc)', se=FALSE, start=list(b=1,cc=1))
#stat_smooth( method="glm", family="poisson", formula='DoS~kW+metier')
# stat_smooth(method="lm")
facet_grid(. ~ metier, scales="free")

update_labels(p, list(x = "kW", y="GrW"))
savePlot(file=file.path(outPath, "plot_DS_GrW_vs_kW_per_metier.png"), type="png")







```