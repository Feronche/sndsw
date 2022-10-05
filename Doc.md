# Documentation

Conversion between raw data in 
(Online DAQ)[https://gitlab.cern.ch/snd-scifi/software/-/wikis/Raw-data-format]


## Raw data to DigiHit 

The conversion is defined in `./sndFairTasks/ConvRawData.cxx`
line 393

```
sipmID = 1000000*int(station[1]-'0') + 100000*orientation + 10000*mat
                                 + 1000*(int(sipmLocal/128)) + chan%128;
```

This is saved in the class `SndlhcHit` as defined in `shipdata/SndlhcHit.h`.

Get parameters with detector ID  

```
SndlhcHit::GetDetectorID() \\ give sipmID
```
This contains where the his is.

Get QDC value with 

```
SndlhcHit::GetSignal(nChan)
```

Get Time of the 

```
SndlhcHit::GetTime(nChan)
```


Hits in SciFi always have

```
Int_t nSiPMs=1,Int_t nSides=0 \\ Get with GetnSiPMs() and GetnSides()
```


Get SiPM position with 

```
geo = SndlhcGeo.GeoInterface(options.geoFile)
A,B = ROOT.TVector3(),ROOT.TVector3()
detID = digi.GetDetectorID()
geo.modules['Scifi'].GetSiPMPosition(detID,A,B) # def in shipLHC/Scifi.cxx

# A and B ==> Left and Right position of a cluster 


# Orientation 
digi.isVertical(): True if `X` else `Y`
```

## Tracking 

Track is reconstructed using either [Kalman Filter](./python/SndlhcTracking.py) or Hough transform 
It uses a cluster defined in `./shipLHC/sndCluster.cxx`

Get Position of a cluster with 

```
cluster = sndCluster()
A,B = ROOT.TVector3(),ROOT.TVector3()
cluster.GetPosition(A,B)
```

Get Z position 

Perhaps 
```
import ROOT
lsOfGlobals = ROOT.gROOT.GetListOfGlobals()
lsOfGlobals.Add(geo.modules['Scifi'])
si = geo.snd_geo.Scifi
```

or get z position from all hits (fit on histogram for each plane) and hardcode

```
geo.modules['Scifi'].GetSiPMPosition(detID,A,B)
Z = A[2]
```

## Loop over multiple files 


Taken from https://github.com/SND-LHC/sndsw/blob/master/shipLHC/scripts/Monitor.py line 135 - 188

```
# Set up TChain
treeName = ""
eventChain = ROOT.TChain(treeName)

# Add files in the run
for _file in glob.glob(PATH_PATTERN):
    eventChain.Add(_file)

# Setup FairANA
run  = ROOT.FairRunAna()
ioman = ROOT.FairRootManager.Instance()
ioman.SetTreeName(eventChain.GetName())

# Def dummy output
outFile = ROOT.TMemFile('dummy','CREATE')

# Setup files in the run
source = ROOT.FairFileSource(eventChain.GetCurrentFile())
for _file in glob.glob(PATH_PATTERN):
    source.AddFile(_file)

# Add source files and set output sink
run.SetSource(source)
sink = ROOT.FairRootFileSink(outFile)
run.SetSink(sink)
```


