---
title: "Triggering with HLT Providers"
teaching: 0
exercises: 25
questions:
- "Why might I need to access the DB conditions for trigger studies?"
- "What are the HLTConfigProvider and HLTPrescaleProvider classes?"
- "What kind of triggers will be needed for our next lessons?"
objectives:
- "Learn why one might want to access additional trigger information from the conditions DB"
- "Learn how to use the HLTConfigProvider and HLTPrescaleProvider classes"
- "Undertand the pre-selection we will be doing for our later analysis"
keypoints:
- "A lot more information related to trigger can be obtained from the HLTConfigProvider and HLTPrescaleProvider and the conditions database"
- "We will be working with pre-filtered data samples later in the workshop"
---

## The HLTConfigProvider and HLTPrescaleProvider

While it is true that one can get most of the trigger information needed directly from the miniAOD files, like we found out in the last episode, there are some cases when this information is not enough.  An example is the case of multi-object triggers.  If one needs to study a trigger in detail it is likely that the [HLTConfigProvider](https://github.com/cms-sw/cmssw/blob/CMSSW_7_6_X/HLTrigger/HLTcore/interface/HLTConfigProvider.h) and the [HLTPrescaleProvider](https://github.com/cms-sw/cmssw/blob/CMSSW_7_6_X/HLTrigger/HLTcore/interface/HLTPrescaleProvider.h) classes are needed. As you can check for yourself these clases have several methods to extract a lot of trigger-related information. Several of those, like the ones related to prescale extraction, need access to the **conditions DB**.  Fortunately, as you remember from yesterday, we already included that to get the *transients tracks* built correctly. We will cover a couple of examples.

## Accessing trigger prescales and acceptance bits

We already have a POET analyzer that uses the above clases.  It has been implemented as `src/TriggerAnalyzer.cc`.  The corresponding module in the `python/poet_cfg.py` is called `mytriggers`.  It has been commented out so for this exercise will uncomment those lines:  After you open the `poet_cfg.py` file and edit it, the snippet should should look like:

~~~
#---- Example on how to add trigger information
#---- To include it, uncomment the lines below and include the
#---- module in the final path
process.mytriggers = cms.EDAnalyzer('TriggerAnalyzer',
                              processName = cms.string("HLT"),
                              #---- These are example of OR of triggers for 2015
                              #---- Wildcards * and ? are accepted (with usual meanings)
                              #---- If left empty, all triggers will run              
                              triggerPatterns = cms.vstring("HLT_IsoMu20_v*","HLT_IsoTkMu20_v*"), 
                              triggerResults = cms.InputTag("TriggerResults","","HLT")
                              )
~~~
{: .language-python}


Note that the configurator accepts sevaral triggers and wildcards.  The combination is an **OR** of the triggers.

Let's include this module in the final paths at the end of the `poet_cfg.py` file.  Let's replace the `mysimpletrig` module so we don't pollute the print out.

~~~
if isData:
	process.p = cms.Path(process.mytriggers+process.hltHighLevel+process.elemufilter+process.myelectrons+process.mymuons+process.mytaus+process.myphotons+process.mypvertex+process.mysimpletrig+
                             process.looseAK4Jets+process.patJetCorrFactorsReapplyJEC+process.slimmedJetsNewJEC+process.myjets+
                             process.looseAK8Jets+process.patJetCorrFactorsReapplyJECAK8+process.slimmedJetsAK8NewJEC+process.myfatjets+
                             process.uncorrectedMet+process.uncorrectedPatMet+process.Type1CorrForNewJEC+process.slimmedMETsNewJEC+process.mymets
                             )
else:
	process.p = cms.Path(process.mytriggers+process.hltHighLevel+process.elemufilter+process.myelectrons+process.mymuons+process.mytaus+process.myphotons+process.mypvertex+process.mysimpletrig+
                             process.mygenparticle+process.looseAK4Jets+process.patJetCorrFactorsReapplyJEC+
                             process.slimmedJetsNewJEC+process.myjets+process.looseAK8Jets+process.patJetCorrFactorsReapplyJECAK8+
                             process.slimmedJetsAK8NewJEC+process.myfatjets+process.uncorrectedMet+process.uncorrectedPatMet+
                             process.Type1CorrForNewJEC+process.slimmedMETsNewJEC+process.mymets
                             )
~~~
{: .language-python}


Let's run POET with the usual `cmsRun python/poet_cfg.py True`.

Let's explore the output, i.e., let's open the `myoutput.root` file.

~~~
root -l myoutput.root
~~~
{: .language-bash}

This is what you will see for the `mytriggers` map *first* and *second*.  The *first* of this map is the name of the trigger and the *second* is the product of $$\text{L1 prescale}\times\text{HLT prescale}\times\text{acceptance bit}$$.


<table align="center">
    <tr>
      <th><a href=""><img src="../fig/trig_first.jpg" style="width: 800px; margin-bottom: 5px;"></a></th>
      <th><a href=""><img src="../fig/trig_second.jpg" style="width: 800px; margin-bottom: 5px;"></a></th>
    </tr>  
</table>

Explore the code and get ready for the last challenge:

> ## Dump the dataset triggers
> 
> As an example of the kind of information you can retrieve using the *HLTConfigProvider* class, try to dump all the triggers that belong to all the datasets for a specific run.
> 
> Some hints:
> * Note that the *HLTConfigProvider* (or its cousin, the *HLTPrescaleProvider*) initialize every run.  This is because the HLT *menu* can change from to run to run, as it was mentioned before.
> * There is a method in the HLTPrescaleProvider class to access the HLTConfigProvider (actually it is used in the code)
> * There is already a method in the HLTConfigProvider to dump (print out) the triggers for each dataset.  It is called `dump` and you can pass the string "Datasets" to it.
> * Don't forget to recompile before running.
>
> > ## Solution
> >
> > Modigy the `src/TriggerAnalyzer.cc` file to add the one-liner `hltPrescaleProvider_.hltConfigProvider().dump("Datasets");` in the `beginRun` function:
> >
> > ~~~
> > // ------------ method called when starting to processes a run  ------------
> > void TriggerAnalyzer::beginRun(edm::Run const& iRun, edm::EventSetup const& iSetup)
> > //--------------------------------------------------------------------------
> > {
> >     using namespace std;
> >     using namespace edm;
> > 
> >     bool changed(true);
> >     hltPrescaleProvider_.init(iRun,iSetup,processName_,changed);
> >     if (changed){
> >       cout<<"HLTConfig has changed for this Run. . . "<<endl;
> >       hltPrescaleProvider_.hltConfigProvider().dump("Datasets");
> >    }
> > } //------------------- beginRun()
> > ~~~
> > {: .language-cpp}
> >
> > This is the final [TriggerAnalyzer.cc](https://raw.githubusercontent.com/cms-opendata-analyses/PhysObjectExtractorTool/odws2022-ttbaljets-prod/PhysObjectExtractor/trunk/TriggerAnalyzer.cc_2) file and the [poet_cfg.py](https://github.com/cms-opendata-analyses/PhysObjectExtractorTool/blob/odws2022-ttbaljets-prod/PhysObjectExtractor/trunk/poet_cfg.py_2) config file.
> {: .solution}
{: .challenge}


## Prepare for the $$t\bar{t}$$ analysis

For Wednesday, due to the large size of the datasets involved, we need to make a pre-filtering or selection to our data.  In this version of POET we have implemented those modifications and apply two filters.  One on the triggers we will be using and a second one to require at least one *tight* electron or one *tight* muon.  Can you spot these filters in the configuration file?

Let's have a look.

The trigger filter:

~~~
#----------- Turn on a trigger filter by adding this module to the the final path below -------#
process.hltHighLevel = cms.EDFilter("HLTHighLevel",
    TriggerResultsTag = cms.InputTag("TriggerResults","","HLT"),
    HLTPaths = cms.vstring('HLT_Ele22_eta2p1_WPLoose_Gsf_v*','HLT_IsoMu20_v*','HLT_IsoTkMu20_v*'),           # provide list of HLT paths (or patterns) you want
    eventSetupPathsKey = cms.string(''), # not empty => use read paths from AlCaRecoTriggerBitsRcd via this key
    andOr = cms.bool(True),             # how to deal with multiple triggers: True (OR) accept if ANY is true, False (AND) accept if ALL are true
    throw = cms.bool(True)    # throw exception on unknown path names
)
~~~
{: .language-python}

You guessed correctly if thought the included triggers are the ones we are going to use for our $$t\bar{t}$ analysis.

The electron/muon filter:

~~~
#---- Example of a very basic home-made filter to select only events of interest
#---- The filter can be added to the running path below if needed but is not applied by default
process.elemufilter = cms.EDFilter('SimpleEleMuFilter',
                                   electrons = cms.InputTag("slimmedElectrons"),
                                   muons = cms.InputTag("slimmedMuons"),
                                   vertices=cms.InputTag("offlineSlimmedPrimaryVertices"),
                                   mu_minpt = cms.double(26),
                                   mu_etacut = cms.double(2.1),
                                   ele_minpt = cms.double(26),
                                   ele_etacut = cms.double(2.1)
                                   )
~~~
{: .language-python}

Finally, let us just mention that, instead of the `mytriggers` module (which was commented out), we have set up a simpler trigger retriever called `mysinpletrig` in the `poet_cfg.py`.

{% include links.md %}

