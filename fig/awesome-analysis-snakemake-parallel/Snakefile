uri = "root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced"

files = [
    "GluGluToHToTauTau",
    "VBF_HToTauTau",
    "DYJetsToLL",
    "DYJetsToLL",
    "TTbar",
    "W1JetsToLNu",
    "W2JetsToLNu",
    "W3JetsToLNu",
    "Run2012B_TauPlusX",
    "Run2012C_TauPlusX",
]

cross_sections = [
    19.6,
    1.55,
    3503.7,
    3503.7,
    225.2,
    6381.2,
    2039.8,
    612.5,
    1.0,
    1.0,
]

short_hands = [
    "ggH",
    "qqH",
    "ZLL",
    "ZTT",
    "TT",
    "W1J",
    "W2J",
    "W3J",
    "dataRunB",
    "dataRunC",
]

rule all:
    input:
        "fit/fit.png",
        "plot/pt_met.png"

rule skim:
    output:
        "skim/{files}_{cross_sections}.root"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master"
    shell:
        "workspace=$(pwd) && mkdir -p skim && cd /analysis/skim && ./skim {uri}/{wildcards.files}.root $workspace/{output} {wildcards.cross_sections} 11467.0 0.1"

rule histogram:
    input:
        "skim/{files}_{cross_sections}.root"
    output:
        "histogram/{files}_{cross_sections}_{short_hands}.root"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master"
    shell:
        "workspace=$(pwd) && mkdir -p histogram && cd /analysis/skim && python histograms.py $workspace/{input} {wildcards.short_hands} $workspace/{output}"

rule merge:
    input:
        expand("histogram/{files}_{cross_sections}_{short_hands}.root", zip, files=files, cross_sections=cross_sections, short_hands=short_hands)
    output:
        "merge/merged.root"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master"
    shell:
        "mkdir -p merge && hadd {output} {input}"

rule fit:
    input:
        "merge/merged.root"
    output:
        "fit/fit.png"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-statistics-stage3:master"
    shell:
        "workspace=$(pwd) && mkdir -p fit && cd /fit && python fit.py $workspace/{input} $workspace/fit"

rule plot:
    input:
        "merge/merged.root"
    output:
        "plot/pt_met.png"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master"
    shell:
        "workspace=$(pwd) && mkdir -p plot && cd /analysis/skim && python plot.py $workspace/{input} $workspace/plot 0.1"
