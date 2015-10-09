# R_load_USGS_streamflow_data

Download data as text file from USGS website
http://waterdata.usgs.gov/nwis/rt
or from links in the kmz file of USGS gages in California
Save webpage as .txt or .csv file.

Webpage and data in txt or csv should look like this:

 ---------------------------------- WARNING ----------------------------------------
 The data you have obtained from this automated U.S. Geological Survey database
 have not received Director's approval and as such are provisional and subject to
 revision.  The data are released on the condition that neither the USGS nor the
 United States Government may be held liable for any damages resulting from its use.
 Additional info: http://waterdata.usgs.gov/nwis/help/?provisional

 File-format description:  http://waterdata.usgs.gov/nwis/?tab_delimited_format_info
 Automated-retrieval info: http://waterdata.usgs.gov/nwis/?automated_retrieval_info

 Contact:   gs-w_support_nwisweb@usgs.gov
 retrieved: 2013-04-20 16:12:06 EDT       (vaww01)

 Data for the following 1 site(s) are contained in this file
    USGS 11010900 WILSON C TRIB NR DULZURA CA
 -----------------------------------------------------------------------------------

 Data provided for site 11010900
    DD parameter statistic   Description
    01   00060     00003     Discharge, cubic feet per second (Mean)

 Data-value qualification codes included in this output: 
     A  Approved for publication -- Processing and review completed. 
     
agency_cd  site_no	datetime	01_00060_00003	01_00060_00003_cd
5s	15s	20d	14n	10s
USGS	11010900	1967-10-01	0.00	A
USGS	11010900	1967-10-02	0.00	A

 ```R

indir = "Complete pathway of working directory goes here"
#  Example:  indir = "G:/SDSU/research/CA/small_watersheds_USGS/discharge_data/daily/"
setwd(indir)

#Gage ID Goes Here
fname = ".csv or txt file name goes here"
#  Example:  fname = "11010900_wilson_creek_trib_daily.txt"

#  METHOD 1:  IF THE NUMBER OF LINES TO SKIP IS THE SAME FOR ALL FILES
x = read.csv(paste(indir,fname,sep=""),skip=27)  # CHANGE SKIP FOR SPECIFIC CVS FILE
  #  COUNT # OF LINES FROM START TO THE FIRST LINE OF DATA--THAT IS THE NUMBER FOR SKIP = XX
dt = strptime(x[,3],format="%m/%d/%Y")   #  Or dt = strptime(g[,3],format="%Y-%m-%d")
  #  Check format of dates in the file.  If m/d/y, use format="%m/%d/%Y".  If y-m-d, use format="%Y-%m-%d"
"Time" =  dt
"Discharge" = x[,4]  
plot(Time,Discharge, type="l")
# ___________________________________________

#  METHOD 2:  LONGER, BUT DON'T NEED TO CHANGE ANY PART OF THIS CODE TO READ DATA PROPERLY
setwd(indir)
GageData = readLines(fname) 
GDNarrow_USGS = (GageData[grep("USGS", GageData)])  # SELECT ONLY LINES WITH "USGS" IN THEM
GDNarrow_dataonly = GDNarrow_USGS[grep("#", GDNarrow_USGS, invert=TRUE)]  # UNSELECT COMMENT LINES
out = strsplit(GDNarrow_dataonly,"\t")  # Change the "\t" to "," if using comma delimited file
g = do.call(rbind,out)
dt = strptime(g[,3],format="%Y-%m-%d")
q = as.numeric(g[,4])
plot(dt,q,type="l") 

#  _______________________________________________

# Aggregate data by calendar year
year = as.numeric(format(dt,"%Y"))
m = as.numeric(format(dt,"%m"))
runoff.ann = aggregate(x[,4],by=list(year),FUN="mean")

# Aggregate by water year
wy = year
wy[m>=10] = wy[m>=10]+1
runoff.ann.wy = aggregate(x[,4],by=list(wy),FUN="mean")

#  Select subset and plot
wyear = c(Year1start,Year1end)
Discharge_Data = x[wy %in% wyear,]
WaterYear = dt[wy %in% wyear]
plot(WaterYear,Discharge_Data[,4],type="l",col="blue") 
