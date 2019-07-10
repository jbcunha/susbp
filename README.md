# susbp
#Seismic Unix Routine for processing SBP Sub Bottom Profiling files

##Seismic Unix sequence to migrate Hull-Mounted SBP files

##Input your sgy file into su environment
##For example if you have an SBP saved as L8.SGY
segyread >l8.su tape=L8.SGY

## To migrate, you need to have delayrt = 0. You can apply this delay with sushift command which will lead to a #trace starting on t=0.0 ms. You may also vanish this value with suchw saving it to apply later. You also need to #have a different cdp value for each shot, which can be obtained with suchw.
suchw <l8Delrt.su key1=cdp key2=tracl a=0 b=1 c=0 d=1 e=1 f=1 >l8DelrtCdp.su

##show headers to check if the change was applied
surange <l8DelrtCdp.su

##Stolt Migration with constant velocity
##You need to know the start cdp, end cdp value and the cdp interval. Also the Nyquist frequency as fmax
sustolt <l8DelrtCdp.su >l8mig1500.su cdpmin=1 cdpmax=15444 dxcdp=0.68 tmig=0.0 vmig=1500.0 fmax=21739 verbose=1

##PLot windowing with suwind and perc=95%
suwind <l8mig1500.su key=tracl min=3000 max=5000 tmin=0.24 tmax=0.34 | suximage perc=95 title="mig 1500m/s"

##You may also plot in wiggle
suxwigb <l8mig1500.su title="migrated"

##To generate output sgy files from the migrated one you need to edit the headers back to the number of samples after #editing with sushift or suchw on delay record time delrt
segyhdrs <l8mig1500.su

##Create a binary.par to editing
bhedtopar < binary outpar=binary.par

##edit the binary.par to have the same number of samples overwriting it
setbhed bfile=binary par=binary.par

##Write the output sgy migrated file
segywrite tape=l8mig1500.sgy <l8mig1500.su
