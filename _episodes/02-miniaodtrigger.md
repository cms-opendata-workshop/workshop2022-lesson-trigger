---
title: "MiniAOD triggering"
teaching: 0
exercises: 25
questions:
- "How can I access trigger information from miniAOD files?"
- "What are trigger objects?"
objectives:
- "Learn how one can retrieve trigger information like pass/fail bits or prescales from miniAOD files"
- "Learn what trigger objects are and why they are important"
keypoints:
- "It is possible to retrieve trigger information from miniAOD files directly"
- "Trigger objects should be check against reconstructed objects"
---

## Introduction

We would like to understand the trigger focusing on our final goal, which is, as you know, to partially reproduce a [$$t\bar{t}$$ analysis](https://link.springer.com/content/pdf/10.1007/JHEP09(2017)051.pdf) from CMS.  

In this episode we will concentrate on extracting the informatio about the trigger and trigger prescales directly from the data stored in the `miniAOD` files.  However, in the next episode, we will attempt to do likewise but using the conditions database instead.  You will learn why these two approaches may be needed.

## MiniAOD triggering

First make sure to fire up your CMSSW Docker container if you haven't already:

~~~
docker start -i my_od #use the name you gave to yours
~~~
{: .language-bash}

Make sure you are at the `/code/CMSSW_7_6_7/src/PhysObjectExtractorTool/PhysObjectExtractor` level, i.e., inside **your poet repository from previous lessons**.

Now, we will check out the version of POET for this lesson, which is the same as the one we have used to **pre-process** the data we will be using later.  `git stash` any lingering modifications or simply do `git checkout .` (note the dot) to reset to the original status of this version.  We will start fresh with a new version, this is the version we use to produce the skmmed datasets with which we will be working tomorrow:

~~~
git checkout odws2022-ttbaljets-prod
~~~
{: .language-bash}

For sanity, let's compile and run POET:

~~~
scram b
cmsRun python/poet_cfg.py True
~~~
{: .language-bash}

Like in the previous lessons, you should see a new `myoutput.root` files emerging.  Let's not worry about it for now, just make sure it is newly produced.

If you look at the `src` directory:

~~~
ls src
~~~
{: .language-bash}

you will see that there is actually an EDAnalyzer called `TriggerAnalyzer.cc`.  There is also a `SimpleTriggerAnalyzer.cc`:  

~~~
ElectronAnalyzer.cc     JetAnalyzer.cc             MuonAnalyzer.cc       SimpleTriggerAnalyzer.cc  TriggerAnalyzer.cc
FatjetAnalyzer.cc       MetAnalyzer.cc             PhotonAnalyzer.cc     TauAnalyzer.cc            VertexAnalyzer.cc
GenParticleAnalyzer.cc  MiniAODTriggerAnalyzer.cc  SimpleEleMuFilter.cc  TriggObjectAnalyzer.cc
~~~
{: .output}

Let's not worry about those right now (we will check them out later) and **pretend there is no analyzer related to extracting the trigger information** at hand.  In fact we do not have an analyzer (yet) that can do what we are about to do: to extract the trigger information directly from the dataset files.  Just for you to remember, we do know the information should be there.  Dump again the content of one of the files we have been working with:

~~~
edmDumpEventContent root://eospublic.cern.ch//eos/opendata/cms/mc/RunIIFall15MiniAODv2/TT_TuneCUETP8M1_13TeV-powheg-pythia8/MINIAODSIM/PU25nsData2015v1_76X_mcRun2_asymptotic_v12_ext3-v1/00000/02837459-03C2-E511-8EA2-002590A887AC.root
~~~
{: .language-bash}

~~~
Type                                  Module                      Label             Process   
----------------------------------------------------------------------------------------------
LHEEventProduct                       "externalLHEProducer"       ""                "LHE"     
GenEventInfoProduct                   "generator"                 ""                "SIM"     
edm::TriggerResults                   "TriggerResults"            ""                "SIM"     
edm::TriggerResults                   "TriggerResults"            ""                "HLT"     
HcalNoiseSummary                      "hcalnoise"                 ""                "RECO"    
L1GlobalTriggerReadoutRecord          "gtDigis"                   ""                "RECO"    
double                                "fixedGridRhoAll"           ""                "RECO"    
double                                "fixedGridRhoFastjetAll"    ""                "RECO"    
double                                "fixedGridRhoFastjetAllCalo"   ""                "RECO"    
double                                "fixedGridRhoFastjetCentral"   ""                "RECO"    
double                                "fixedGridRhoFastjetCentralCalo"   ""                "RECO"    
double                                "fixedGridRhoFastjetCentralChargedPileUp"   ""                "RECO"    
double                                "fixedGridRhoFastjetCentralNeutral"   ""                "RECO"    
edm::TriggerResults                   "TriggerResults"            ""                "RECO"    
reco::BeamHaloSummary                 "BeamHaloSummary"           ""                "RECO"    
reco::BeamSpot                        "offlineBeamSpot"           ""                "RECO"    
vector<l1extra::L1EmParticle>         "l1extraParticles"          "Isolated"        "RECO"    
vector<l1extra::L1EmParticle>         "l1extraParticles"          "NonIsolated"     "RECO"    
vector<l1extra::L1EtMissParticle>     "l1extraParticles"          "MET"             "RECO"    
vector<l1extra::L1EtMissParticle>     "l1extraParticles"          "MHT"             "RECO"    
vector<l1extra::L1HFRings>            "l1extraParticles"          ""                "RECO"    
vector<l1extra::L1JetParticle>        "l1extraParticles"          "Central"         "RECO"    
vector<l1extra::L1JetParticle>        "l1extraParticles"          "Forward"         "RECO"    
vector<l1extra::L1JetParticle>        "l1extraParticles"          "IsoTau"          "RECO"    
vector<l1extra::L1JetParticle>        "l1extraParticles"          "Tau"             "RECO"    
vector<l1extra::L1MuonParticle>       "l1extraParticles"          ""                "RECO"    
edm::SortedCollection<EcalRecHit,edm::StrictWeakOrdering<EcalRecHit> >    "reducedEgamma"             "reducedEBRecHits"   "PAT"     
edm::SortedCollection<EcalRecHit,edm::StrictWeakOrdering<EcalRecHit> >    "reducedEgamma"             "reducedEERecHits"   "PAT"     
edm::SortedCollection<EcalRecHit,edm::StrictWeakOrdering<EcalRecHit> >    "reducedEgamma"             "reducedESRecHits"   "PAT"     
edm::TriggerResults                   "TriggerResults"            ""                "PAT"     
edm::ValueMap<float>                  "offlineSlimmedPrimaryVertices"   ""                "PAT"     
pat::PackedTriggerPrescales           "patTrigger"                ""                "PAT"     
pat::PackedTriggerPrescales           "patTrigger"                "l1max"           "PAT"     
pat::PackedTriggerPrescales           "patTrigger"                "l1min"           "PAT"     
vector<PileupSummaryInfo>             "slimmedAddPileupInfo"      ""                "PAT"     
vector<pat::Electron>                 "slimmedElectrons"          ""                "PAT"     
vector<pat::Jet>                      "slimmedJets"               ""                "PAT"     
vector<pat::Jet>                      "slimmedJetsAK8"            ""                "PAT"     
vector<pat::Jet>                      "slimmedJetsPuppi"          ""                "PAT"     
vector<pat::Jet>                      "slimmedJetsAK8PFCHSSoftDropPacked"   "SubJets"         "PAT"     
vector<pat::Jet>                      "slimmedJetsCMSTopTagCHSPacked"   "SubJets"         "PAT"     
vector<pat::MET>                      "slimmedMETs"               ""                "PAT"     
vector<pat::MET>                      "slimmedMETsPuppi"          ""                "PAT"     
vector<pat::Muon>                     "slimmedMuons"              ""                "PAT"     
vector<pat::PackedCandidate>          "lostTracks"                ""                "PAT"     
vector<pat::PackedCandidate>          "packedPFCandidates"        ""                "PAT"     
vector<pat::PackedGenParticle>        "packedGenParticles"        ""                "PAT"     
vector<pat::Photon>                   "slimmedPhotons"            ""                "PAT"     
vector<pat::Tau>                      "slimmedTaus"               ""                "PAT"     
vector<pat::TriggerObjectStandAlone>    "selectedPatTrigger"        ""                "PAT"     
vector<reco::CATopJetTagInfo>         "caTopTagInfosPAT"          ""                "PAT"     
vector<reco::CaloCluster>             "reducedEgamma"             "reducedEBEEClusters"   "PAT"     
vector<reco::CaloCluster>             "reducedEgamma"             "reducedESClusters"   "PAT"     
vector<reco::Conversion>              "reducedEgamma"             "reducedConversions"   "PAT"     
vector<reco::Conversion>              "reducedEgamma"             "reducedSingleLegConversions"   "PAT"     
vector<reco::GenJet>                  "slimmedGenJets"            ""                "PAT"     
vector<reco::GenJet>                  "slimmedGenJetsAK8"         ""                "PAT"     
vector<reco::GenParticle>             "prunedGenParticles"        ""                "PAT"     
vector<reco::GsfElectronCore>         "reducedEgamma"             "reducedGedGsfElectronCores"   "PAT"     
vector<reco::PhotonCore>              "reducedEgamma"             "reducedGedPhotonCores"   "PAT"     
vector<reco::SuperCluster>            "reducedEgamma"             "reducedSuperClusters"   "PAT"     
vector<reco::Vertex>                  "offlineSlimmedPrimaryVertices"   ""                "PAT"     
vector<reco::VertexCompositePtrCandidate>    "slimmedSecondaryVertices"   ""                "PAT"
~~~
{: .output}

Clearly, there are `selectedPatTrigger` and `patTrigger` collections there.

Although we do not have such analyzer yet, we know where to find information on how to implement it.  If we go to the [WorkBookMiniAOD2015](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookMiniAOD2015#Trigger) CMS Twiki page, we will find the relevant information necessary to our purposes.  Let's have a look.

As you can see, an example is already provided on how to build such an analyzer with its corresponding configuration.

We have copied the analyzer example here for you:

~~~
// system include files
#include <memory>
#include <cmath>

// user include files
#include "FWCore/Framework/interface/Frameworkfwd.h"
#include "FWCore/Framework/interface/EDAnalyzer.h"
#include "FWCore/Framework/interface/Event.h"
#include "FWCore/Framework/interface/MakerMacros.h"
#include "FWCore/ParameterSet/interface/ParameterSet.h"

#include "DataFormats/Math/interface/deltaR.h"
#include "FWCore/Common/interface/TriggerNames.h"
#include "DataFormats/Common/interface/TriggerResults.h"
#include "DataFormats/PatCandidates/interface/TriggerObjectStandAlone.h"
#include "DataFormats/PatCandidates/interface/PackedTriggerPrescales.h"

class MiniAODTriggerAnalyzer : public edm::EDAnalyzer {
   public:
      explicit MiniAODTriggerAnalyzer(const edm::ParameterSet&);
      ~MiniAODTriggerAnalyzer() {}

   private:
      virtual void analyze(const edm::Event&, const edm::EventSetup&) override;

      edm::EDGetTokenT<edm::TriggerResults> triggerBits_;
      edm::EDGetTokenT<pat::TriggerObjectStandAloneCollection> triggerObjects_;
      edm::EDGetTokenT<pat::PackedTriggerPrescales> triggerPrescales_;
};

MiniAODTriggerAnalyzer::MiniAODTriggerAnalyzer(const edm::ParameterSet& iConfig):
    triggerBits_(consumes<edm::TriggerResults>(iConfig.getParameter<edm::InputTag>("bits"))),
    triggerObjects_(consumes<pat::TriggerObjectStandAloneCollection>(iConfig.getParameter<edm::InputTag>("objects"))),
    triggerPrescales_(consumes<pat::PackedTriggerPrescales>(iConfig.getParameter<edm::InputTag>("prescales")))
{
}

void MiniAODTriggerAnalyzer::analyze(const edm::Event& iEvent, const edm::EventSetup& iSetup)
{
    edm::Handle<edm::TriggerResults> triggerBits;
    edm::Handle<pat::TriggerObjectStandAloneCollection> triggerObjects;
    edm::Handle<pat::PackedTriggerPrescales> triggerPrescales;

    iEvent.getByToken(triggerBits_, triggerBits);
    iEvent.getByToken(triggerObjects_, triggerObjects);
    iEvent.getByToken(triggerPrescales_, triggerPrescales);

    const edm::TriggerNames &names = iEvent.triggerNames(*triggerBits);
    std::cout << "\n === TRIGGER PATHS === " << std::endl;
    for (unsigned int i = 0, n = triggerBits->size(); i < n; ++i) {
        std::cout << "Trigger " << names.triggerName(i) << 
                ", prescale " << triggerPrescales->getPrescaleForIndex(i) <<
                ": " << (triggerBits->accept(i) ? "PASS" : "fail (or not run)") 
                << std::endl;
    }
    std::cout << "\n === TRIGGER OBJECTS === " << std::endl;
    for (pat::TriggerObjectStandAlone obj : *triggerObjects) { // note: not "const &" since we want to call unpackPathNames
        obj.unpackPathNames(names);
        std::cout << "\tTrigger object:  pt " << obj.pt() << ", eta " << obj.eta() << ", phi " << obj.phi() << std::endl;
        // Print trigger object collection and type
        std::cout << "\t   Collection: " << obj.collection() << std::endl;
        std::cout << "\t   Type IDs:   ";
        for (unsigned h = 0; h < obj.filterIds().size(); ++h) std::cout << " " << obj.filterIds()[h] ;
        std::cout << std::endl;
        // Print associated trigger filters
        std::cout << "\t   Filters:    ";
        for (unsigned h = 0; h < obj.filterLabels().size(); ++h) std::cout << " " << obj.filterLabels()[h];
        std::cout << std::endl;
        std::vector<std::string> pathNamesAll  = obj.pathNames(false);
        std::vector<std::string> pathNamesLast = obj.pathNames(true);
        // Print all trigger paths, for each one record also if the object is associated to a 'l3' filter (always true for the
        // definition used in the PAT trigger producer) and if it's associated to the last filter of a successfull path (which
        // means that this object did cause this trigger to succeed; however, it doesn't work on some multi-object triggers)
        std::cout << "\t   Paths (" << pathNamesAll.size()<<"/"<<pathNamesLast.size()<<"):    ";
        for (unsigned h = 0, n = pathNamesAll.size(); h < n; ++h) {
            bool isBoth = obj.hasPathName( pathNamesAll[h], true, true ); 
            bool isL3   = obj.hasPathName( pathNamesAll[h], false, true ); 
            bool isLF   = obj.hasPathName( pathNamesAll[h], true, false ); 
            bool isNone = obj.hasPathName( pathNamesAll[h], false, false ); 
            std::cout << "   " << pathNamesAll[h];
            if (isBoth) std::cout << "(L,3)";
            if (isL3 && !isBoth) std::cout << "(*,3)";
            if (isLF && !isBoth) std::cout << "(L,*)";
            if (isNone && !isBoth && !isL3 && !isLF) std::cout << "(*,*)";
        }
        std::cout << std::endl;
    }
    std::cout << std::endl;

}

//define this as a plug-in
DEFINE_FWK_MODULE(MiniAODTriggerAnalyzer);
~~~
{: .language-cpp}


And the configuration file from where we can snatch a valid snippet for our stolen analyzer:

~~~
import FWCore.ParameterSet.Config as cms

process = cms.Process("Demo")

process.load("FWCore.MessageService.MessageLogger_cfi")
process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(10) )

process.source = cms.Source("PoolSource",
    fileNames = cms.untracked.vstring(
        '/store/cmst3/user/gpetrucc/miniAOD/v1/TT_Tune4C_13TeV-pythia8-tauola_PU_S14_PAT.root'
    )
)

process.demo = cms.EDAnalyzer("MiniAODTriggerAnalyzer",
    bits = cms.InputTag("TriggerResults","","HLT"),
    prescales = cms.InputTag("patTrigger"),
    objects = cms.InputTag("selectedPatTrigger"),
)

process.p = cms.Path(process.demo)
~~~
{: .language-python}


Let's do that, let's create a `src/MiniAODTriggerAnalyzer.cc` from scratch.  From your local directory just open up your favorite editor and copy paste the code.  Save it and compile with `scram b`, as always.

Now, let's add the python module that we need to configure it to our `python/poet_cfg.py` file.  We will call it `myminiaodtrig` instead of `demo`.  From you local directory open the `python/poet_cfg.py` configurator and add this module just before the `process.simpletrig` module (this is just to keep things organized).  that portion of the config files should look like

~~~
#---- Example on how to add trigger information
#---- To include it, uncomment the lines below and include the
#---- module in the final path
#process.mytriggers = cms.EDAnalyzer('TriggerAnalyzer',
#                              processName = cms.string("HLT"),
#                              #---- These are example of OR of triggers for 2015
#                              #---- Wildcards * and ? are accepted (with usual meanings)
#                              #---- If left empty, all triggers will run              
#                              triggerPatterns = cms.vstring("HLT_IsoMu20_v*","HLT_IsoTkMu20_v*"), 
#                              triggerResults = cms.InputTag("TriggerResults","","HLT")
#                              )

#---- test to see if we get trigger info ----
process.myminiaodtrig = cms.EDAnalyzer("MiniAODTriggerAnalyzer",
    bits = cms.InputTag("TriggerResults","","HLT"),
    prescales = cms.InputTag("patTrigger"),
    objects = cms.InputTag("selectedPatTrigger"),
)

#------------Example of simple trigger module with parameters by hand-------------------#
process.mysimpletrig = cms.EDAnalyzer('SimpleTriggerAnalyzer',
                              processName = cms.string("HLT"),
                              triggerResults = cms.InputTag("TriggerResults","","HLT")
                              )
~~~
{: .language-python}

Now let's make sure it runs in the CMSSW path at the end of the configuration file.  Let's just add it to both options Data or MC.  Make sure you add it to the beginning of the sequence.  This is because we need to avoid the intial filters that are already in place (more on this later).

~~~
if isData:
	process.p = cms.Path(process.myminiaodtrig+process.hltHighLevel+process.elemufilter+process.myelectrons+process.mymuons+process.mytaus+process.myphotons+process.mypvertex+process.mysimpletrig+
                             process.looseAK4Jets+process.patJetCorrFactorsReapplyJEC+process.slimmedJetsNewJEC+process.myjets+
                             process.looseAK8Jets+process.patJetCorrFactorsReapplyJECAK8+process.slimmedJetsAK8NewJEC+process.myfatjets+
                             process.uncorrectedMet+process.uncorrectedPatMet+process.Type1CorrForNewJEC+process.slimmedMETsNewJEC+process.mymets
                             )
else:
	process.p = cms.Path(process.myninioaodtrig+process.hltHighLevel+process.elemufilter+process.myelectrons+process.mymuons+process.mytaus+process.myphotons+process.mypvertex+process.mysimpletrig+
                             process.mygenparticle+process.looseAK4Jets+process.patJetCorrFactorsReapplyJEC+
                             process.slimmedJetsNewJEC+process.myjets+process.looseAK8Jets+process.patJetCorrFactorsReapplyJECAK8+
                             process.slimmedJetsAK8NewJEC+process.myfatjets+process.uncorrectedMet+process.uncorrectedPatMet+
                             process.Type1CorrForNewJEC+process.slimmedMETsNewJEC+process.mymets
                             )
~~~
{: .language-python}

Lets do just one more change before we get to run POET.  Let's print out for each event, we can achive that by changing the `MessageLogger` lines to:

~~~
#---- Configure the framework messaging system
#---- https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideMessageLogger
process.load("FWCore.MessageService.MessageLogger_cfi")
#process.MessageLogger.cerr.threshold = "WARNING"
#process.MessageLogger.categories.append("POET")
#process.MessageLogger.cerr.INFO = cms.untracked.PSet(
#    limit=cms.untracked.int32(-1))
#process.options = cms.untracked.PSet(wantSummary=cms.untracked.bool(True))
process.MessageLogger.cerr.FwkReport.reportEvery = 1
~~~
{: .language-python}

I.e., comment out most of the lines leaving on only the first one and adding the `process.MessageLogger.cerr.FwkReport.reportEvery = 1` line.


Now, let's run with `cmsRun python/poet_cfg.py True`.  We will get a lot of print-outs. Let's check the printout for the last event:

> ## Trigger dump from MiniAOD
> 
> > ## View dump
> >
> > ~~~
> > Begin processing the 100th record. Run 259809, Event 167531933, LumiSection 132 at 31-Jul-2022 18:03:21.705 CEST
> > 
> >  === TRIGGER PATHS === 
> > Trigger HLTriggerFirstPath, prescale 1: fail (or not run)
> > Trigger HLT_AK8PFJet360_TrimMass30_v3, prescale 1: fail (or not run)
> > Trigger HLT_AK8PFHT700_TrimR0p1PT0p03Mass50_v3, prescale 1: fail (or not run)
> > Trigger HLT_AK8PFHT650_TrimR0p1PT0p03Mass50_v2, prescale 1: fail (or not run)
> > Trigger HLT_AK8PFHT600_TrimR0p1PT0p03Mass50_BTagCSV0p45_v2, prescale 1: fail (or not run)
> > Trigger HLT_CaloJet500_NoJetID_v2, prescale 1: fail (or not run)
> > Trigger HLT_Dimuon13_PsiPrime_v2, prescale 1: fail (or not run)
> > Trigger HLT_Dimuon13_Upsilon_v2, prescale 1: fail (or not run)
> > Trigger HLT_Dimuon20_Jpsi_v2, prescale 1: fail (or not run)
> > Trigger HLT_DoubleEle24_22_eta2p1_WPLoose_Gsf_v2, prescale 1: fail (or not run)
> > Trigger HLT_DoubleEle33_CaloIdL_GsfTrkIdVL_MW_v3, prescale 1: fail (or not run)
> > Trigger HLT_DoubleEle33_CaloIdL_GsfTrkIdVL_v3, prescale 1: fail (or not run)
> > Trigger HLT_DoubleMediumIsoPFTau35_Trk1_eta2p1_Reg_v2, prescale 1: fail (or not run)
> > Trigger HLT_DoubleMediumIsoPFTau40_Trk1_eta2p1_Reg_v4, prescale 1: fail (or not run)
> > Trigger HLT_DoubleMu33NoFiltersNoVtx_v2, prescale 1: fail (or not run)
> > Trigger HLT_DoubleMu38NoFiltersNoVtx_v2, prescale 1: fail (or not run)
> > Trigger HLT_DoubleMu23NoFiltersNoVtxDisplaced_v2, prescale 1: fail (or not run)
> > Trigger HLT_DoubleMu28NoFiltersNoVtxDisplaced_v2, prescale 1: fail (or not run)
> > Trigger HLT_DoubleMu4_3_Bs_v2, prescale 1: fail (or not run)
> > Trigger HLT_DoubleMu4_3_Jpsi_Displaced_v2, prescale 1: fail (or not run)
> > .
> > .
> > .
> > === TRIGGER OBJECTS === 
> > 	Trigger object:  pt 23.8237, eta -1.59244, phi 0.781614
> > 	   Collection: hltHighPtTkMuonCands::HLT
> > 	   Type IDs:    83
> > 	   Filters:     hltL3fL1sMu16L1f0Tkf18QL3OldCalotrkIsoFiltered0p09 hltL3fL1sMu16L1f0Tkf18QL3trkIsoFiltered0p09 hltL3fL1sMu16L1f0Tkf20QL3trkIsoFiltered0p09 hltL3fL1sMu16f0TkFiltered18Q hltL3fL1sMu16f0TkFiltered18QL3OldpfhcalIsoRhoFilteredHB0p21HE0p22 hltL3fL1sMu16f0TkFiltered18QL3pfOldecalIsoRhoFilteredEB0p11EE0p08 hltL3fL1sMu16f0TkFiltered18QL3pfecalIsoRhoFilteredEB0p11EE0p08 hltL3fL1sMu16f0TkFiltered18QL3pfhcalIsoRhoFilteredHB0p21HE0p22 hltL3fL1sMu16f0TkFiltered20Q hltL3fL1sMu16f0TkFiltered20QL3pfecalIsoRhoFilteredEB0p11EE0p08 hltL3fL1sMu16f0TkFiltered20QL3pfhcalIsoRhoFilteredHB0p21HE0p22 hltL3fL1sMu20L1f0Tkf22QL3trkIsoFiltered0p09 hltL3fL1sMu20f0TkFiltered22Q hltL3fL1sMu20f0TkFiltered22QL3pfecalIsoRhoFilteredEB0p11EE0p08 hltL3fL1sMu20f0TkFiltered22QL3pfhcalIsoRhoFilteredHB0p21HE0p22
> > 	   Paths (4/4):       HLT_OldIsoTkMu18_v2(L,3)   HLT_IsoTkMu18_v2(L,3)   HLT_IsoTkMu20_v4(L,3)   HLT_IsoTkMu22_v2(L,3)
> > 	Trigger object:  pt 34.9, eta -1.598, phi 0.766604
> > 	   Collection: hltL2MuonCandidates::HLT
> > 	   Type IDs:    83
> > 	   Filters:     hltL2fL1sMu14erorMu16L1f0L2Filtered0 hltL2fL1sMu16Eta2p1L1f0L2Filtered10Q hltL2fL1sMu16L1f0L2Filtered10Q hltL2fL1sMu16erorMu20erL1f0L2Filtered16Q hltL2fL1sMu16orMu20erorMu25L1f0L2Filtered0 hltL2fL1sMu16orMu25L1f0L2Filtered10Q hltL2fL1sMu16orMu25L1f0L2Filtered16Q hltL2fL1sMu16orMu25L1f0L2Filtered25 hltL2fL1sMu20L1f0L2Filtered10Q hltL2fL1sMu25L1f0L2Filtered10Q hltL2fL1sSingleMu16erL1f0L2Filtered10Q
> > 	   Paths (4/0):       HLT_IsoMu18_v2(*,3)   HLT_OldIsoMu18_v1(*,3)   HLT_IsoMu20_v3(*,3)   HLT_IsoMu22_v2(*,3)
> > 	Trigger object:  pt 37.6634, eta -1.59606, phi 0.757304
> > 	   Collection: hltL2MuonCandidatesNoVtx::HLT
> > 	   Type IDs:    83
> > 	   Filters:     hltL2fL1sMu16orMu25L1f0L2NoVtxFiltered16
> > 	   Paths (0/0):    
> > 	Trigger object:  pt 23.7557, eta -1.59244, phi 0.781631
> > 	   Collection: hltL3MuonCandidates::HLT
> > 	   Type IDs:    83
> > 	   Filters:     hltL3crIsoL1sMu16Eta2p1L1f0L2f10QL3f20QL3trkIsoFiltered0p09 hltL3crIsoL1sMu16L1f0L2f10QL3f18QL3OldCaloIsotrkIsoFiltered0p09 hltL3crIsoL1sMu16L1f0L2f10QL3f18QL3trkIsoFiltered0p09 hltL3crIsoL1sMu16L1f0L2f10QL3f20QL3trkIsoFiltered0p09 hltL3crIsoL1sMu20L1f0L2f10QL3f22QL3trkIsoFiltered0p09 hltL3crIsoL1sSingleMu16erL1f0L2f10QL3f17QL3pfecalIsoRhoFilteredEB0p11EE0p08 hltL3crIsoL1sSingleMu16erL1f0L2f10QL3f17QL3pfhcalIsoRhoFilteredHB0p21HE0p22 hltL3crIsoL1sSingleMu16erL1f0L2f10QL3f17QL3trkIsoFiltered0p09 hltL3fL1sMu14erorMu16L1f0L2f0L3Filtered16 hltL3fL1sMu16Eta2p1L1f0L2f10QL3Filtered20Q hltL3fL1sMu16Eta2p1L1f0L2f10QL3Filtered20QL3pfecalIsoRhoFilteredEB0p11EE0p08 hltL3fL1sMu16Eta2p1L1f0L2f10QL3Filtered20QL3pfhcalIsoRhoFilteredHB0p21HE0p22 hltL3fL1sMu16L1f0L2f10QL3Filtered18Q hltL3fL1sMu16L1f0L2f10QL3Filtered18QL3pfecalIsoRhoFilteredEB0p11EE0p08 hltL3fL1sMu16L1f0L2f10QL3Filtered18QL3pfecalOldIsoRhoFilteredEB0p11EE0p08 hltL3fL1sMu16L1f0L2f10QL3Filtered18QL3pfhcalIsoRhoFilteredHB0p21HE0p22 hltL3fL1sMu16L1f0L2f10QL3Filtered18QL3pfhcalOldIsoRhoFilteredHB0p21HE0p22 hltL3fL1sMu16L1f0L2f10QL3Filtered20Q hltL3fL1sMu16L1f0L2f10QL3Filtered20QL3pfecalIsoRhoFilteredEB0p11EE0p08 hltL3fL1sMu16L1f0L2f10QL3Filtered20QL3pfhcalIsoRhoFilteredHB0p21HE0p22 hltL3fL1sMu20L1f0L2f10QL3Filtered22Q hltL3fL1sMu20L1f0L2f10QL3Filtered22QL3pfecalIsoRhoFilteredEB0p11EE0p08 hltL3fL1sMu20L1f0L2f10QL3Filtered22QL3pfhcalIsoRhoFilteredHB0p21HE0p22 hltL3fL1sSingleMu16erL1f0L2f10QL3Filtered17Q
> > 	   Paths (4/4):       HLT_OldIsoMu18_v1(L,3)   HLT_IsoMu18_v2(L,3)   HLT_IsoMu20_v3(L,3)   HLT_IsoMu22_v2(L,3)
> > 	Trigger object:  pt 40, eta -1.55, phi 0.698132
> > 	   Collection: hltL1extraParticles::HLT
> > 	   Type IDs:    -81
> > 	   Filters:     hltL1sAlCaRPC hltL1sL1SingleMu14erORSingleMu16 hltL1sL1SingleMu16 hltL1sL1SingleMu16ORSingleMu20erORSingleMu25 hltL1sL1SingleMu16ORSingleMu25 hltL1sL1SingleMu16er hltL1sL1SingleMu16erORSingleMu20er hltL1sL1SingleMu20 hltL1sL1SingleMu20er hltL1sL1SingleMu25
> > 	   Paths (8/0):       HLT_IsoMu18_v2(*,3)   HLT_OldIsoMu18_v1(*,3)   HLT_IsoMu20_v3(*,3)   HLT_IsoTkMu18_v2(*,3)   HLT_OldIsoTkMu18_v2(*,3)   HLT_IsoTkMu20_v4(*,3)   HLT_IsoMu22_v2(*,3)   HLT_IsoTkMu22_v2(*,3)
> > 
31-Jul-2022 18:03:21 CEST  Closed file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleMuon/MINIAOD/16Dec2015-v1/10000/00006301-CAA8-E511-AD39-549F35AD8BC9.root
> > 
> > ~~~
> > {: .output}
> >
> > Here are the final [MiniAODTriggerAnalyzer.cc](https://raw.githubusercontent.com/cms-opendata-analyses/PhysObjectExtractorTool/odws2022-ttbaljets-prod/PhysObjectExtractor/trunk/MiniAODTriggerAnalyzer.cc_1) and [poet_cfg.py](https://raw.githubusercontent.com/cms-opendata-analyses/PhysObjectExtractorTool/odws2022-ttbaljets-prod/PhysObjectExtractor/trunk/poet_cfg.py_1) files.
> > 
> {: .solution}
{: .challenge}

So, we not only get the triggers and their **pass/fail** status, but also the prescales and the possible **trigger objects** associated with them (associated with their trigger filters).

### Trigger objects

Let's explore the code which prints the last part  to understand what it means.  The HLT is like a reconstruction algorithm capable of identifying particles very very fast.  Therefore, an HLT muon, for example, is a trigger object that has a four momentum.  These trigger objects are associated to, commonly,  the last filter executed by a given path in the HLT.  Usually one wants to check whether a given particle that is being used in a given analysis coincides with the trigger object that fired the trigger (this is called **trigger matching**).  We won't go into more details about that.

~~~
std::cout << "\n === TRIGGER OBJECTS === " << std::endl;
    for (pat::TriggerObjectStandAlone obj : *triggerObjects) { // note: not "const &" since we want to call unpackPathNames
        obj.unpackPathNames(names);
        std::cout << "\tTrigger object:  pt " << obj.pt() << ", eta " << obj.eta() << ", phi " << obj.phi() << std::endl;
        // Print trigger object collection and type
        std::cout << "\t   Collection: " << obj.collection() << std::endl;
        std::cout << "\t   Type IDs:   ";
        for (unsigned h = 0; h < obj.filterIds().size(); ++h) std::cout << " " << obj.filterIds()[h] ;
        std::cout << std::endl;
        // Print associated trigger filters
        std::cout << "\t   Filters:    ";
        for (unsigned h = 0; h < obj.filterLabels().size(); ++h) std::cout << " " << obj.filterLabels()[h];
        std::cout << std::endl;
        std::vector<std::string> pathNamesAll  = obj.pathNames(false);
        std::vector<std::string> pathNamesLast = obj.pathNames(true);
        // Print all trigger paths, for each one record also if the object is associated to a 'l3' filter (always true for the
        // definition used in the PAT trigger producer) and if it's associated to the last filter of a successfull path (which
        // means that this object did cause this trigger to succeed; however, it doesn't work on some multi-object triggers)
        std::cout << "\t   Paths (" << pathNamesAll.size()<<"/"<<pathNamesLast.size()<<"):    ";
        for (unsigned h = 0, n = pathNamesAll.size(); h < n; ++h) {
            bool isBoth = obj.hasPathName( pathNamesAll[h], true, true ); 
            bool isL3   = obj.hasPathName( pathNamesAll[h], false, true ); 
            bool isLF   = obj.hasPathName( pathNamesAll[h], true, false ); 
            bool isNone = obj.hasPathName( pathNamesAll[h], false, false ); 
            std::cout << "   " << pathNamesAll[h];
            if (isBoth) std::cout << "(L,3)";
            if (isL3 && !isBoth) std::cout << "(*,3)";
            if (isLF && !isBoth) std::cout << "(L,*)";
            if (isNone && !isBoth && !isL3 && !isLF) std::cout << "(*,*)";
        }
        std::cout << std::endl;
    }
~~~
{: .language-cpp}


![](../fig/triggerpath.png)


Finally, note that we are not writing any information into our `myoutput.root` file.  You are welcome to work with us if you would like to help us implement this in the `2015` version of POET.

{% include links.md %}

