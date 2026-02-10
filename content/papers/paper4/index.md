---
title: "Alphafold: A Deep Dive into Architecture, Intuition and Implementation" 
date: 2025-10-29
tags: ["protein structure prediction","alphafold","deep learning","protein modeling","protein folding","protein design","protein engineering","protein dynamics","protein stability","protein function"]
author: ["Jebrai J."]
description: "This markdown document explores the architecture and intuition behind AlphaFold and other. It is not by any means the final product as of december 2025." 
summary: "This paper provides a comprehensive overview of the AlphaFold model, a deep learning model (Aas well as other models) for predicting protein structures. It explores the motivation behind each idea, problem statements, and solutions, as well as potential extensions and questions for further exploration."
cover:
    image: "paper4.png"
    alt:
    relative: true
editPost:
    URL:
    Text: 
---
# Scope

This document should serve anyone with curiosity related to the following, as we'll cover: 

- Basic protein knowledge (no promises),
- The field of "proteins" as a whole, as well as providing potential elaboration ingestion in case you'll be delving further in that field, 
- The Proteins Problem Statement(s) along with their solutions, by fitting the (refined) data to a (relatively custom) model,
- Core concepts of the architecture,  
- And lastly what could be used to extend the idea itself of AlphaFold 1, as well as questions that have incurred to me personally while attempting to understand the architecture motivation and the interconnected nature of everything. These are thrown throughout the document as "insights" and in the (relatively) simple methodology of demonstration itself. However, it is favorable for the reader to absorb it however they like. 

At the [Engineering](#engineering) section, everything is wrapped with a "before in each component pass forward". There is also a [Building up from the smallest to the biggest protein component](#building-up-from-the-smallest-to-the-biggest-protein-component) that serves a bigger picture on what we'll be dealing with for later stages, as well as solidify our understanding towards each components role.
### Extension

While there will be a thorough study towards AlphaFold 2 and other architectures, some of the notes will be left as possibilities for extension for oneself to exercise the muscle of "How can we improve this?" and "What is this prototype doing inefficiently?", prompting the reader to delve into the material and realize they're potentially passing an important, guaranteed questions that are answered in later architectures (something that is built upon; AF2 and AF3 for example and others).

The reason (we) chose AlphaFold 1 (with extension potential mentioned in [Foreword](#foreword)) and not its modern variants (AlphaFold 2's breakthrough) is due to the foundational approach towards the problem at hand.

>*You'd notice that a prototype architecture would contain several distinct modules, similar to the idea of "making it happen and perfect it later"*

All intentions here lie on gaining intuition that will matter later on either on the same exact task and the elaborative extensions, or the in different disciplines (the science of adaptability). Hopefully that will serve as your "purpose" in reading the document at hand.

**Considerations:** 

- This may be published before polished, maybe even before folding completely; meaning that some text are also thoughts immediately jotted down, which will be separated from practicality with each revision.
- ***Some images are clickable and have links in them***. Be careful clicking any links before the final revision is out (HTTP etc.). 
- Some sources are also linked partially or fully, even though they're not the entire source in many of the text linking it.
- The document may be separated due to how large it is currently, but there will be attempts in making them fit to keep it holistic and central. 

*Any feedback of any form is welcome.*
## Brevity

To avoid repeating "AlphaFold X by DeepMind"; AFX is the norm,
AminoAcid abbreviation of "AA" which is used synonymously with "residue" or "res",
along with the abbreviation of Angstroms "Å": "Ang."
Reinforcement Learning: "RL"
[ProtienDataBank](https://www.rcsb.org/#Category-deposit): "PDB"
....
Other abbreviations are made on-spot.
There are also cases for alternating between them for clarity and intuition.

## Foreword

There are many architectures that have been placed in the record for the "protein folding" problem:

- AlphaFold 1, which is our main topic; AlphaFold 1 (AF1) basically did:
  'Alphafold uses advanced biotechnology and AI to help determine ***longer*** protein structures over a ***shorter*** period)'. This is the simplest algorithm (least devouring of data as well as the complexity threshold), but is the most non end-to-end "solution" (several components and "externalities"), which also lays groundwork and first-hand intuition on the problem statement.
- AlphaFold 2 and its extensions: 2.3, and multimer which emphasizes a minor role in protein-protein interactions (some improved IDRs capturing capability), but it did well in the static protein-protein complexes (see [AlphaFold | EMBL-EBI Training](https://www.ebi.ac.uk/training/online/courses/alphafold/) as well as for later purposes, [this link here](https://elearning.vib.be/courses/alphafold/lessons/the-alphafold-pipeline/topic/overview-of-the-architecture/)).
- AlphaFold 3, which is abit out of the current scope: Prediction of interactions not just of proteins, but also DNA, RNA, small molecules (ligands), ions and chemical modifications.

### Purpose

Understanding why proteins fold is important towards understanding [the problem statement](#the-problem-statement), along with the motive of this entire interaction between the five fields (Biochemistry in the interactions, Bioinformatics and genetics relating to MSAs, Artificial intelligence (attention, Evo-former, diffusion, etc), Structural Biology as in the decades-old dataset of [PDB](#pdb) and experimental methods. And finally; Medicine and Biotechnology, **which are currently the primary beneficiaries** (think of it as the biggest "To-dos" after a breakthrough), using the insights derived from the other fields to:

1) **Accelerate drug discovery.** Drugs typically work by binding to specific protein targets in the body (a receptor on a cell, or an enzyme in a pathogen, or ). So if we know the precise 3D structure of a target protein, we can design small-molecule drugs that fit perfectly into its active site see [this](https://pmc.ncbi.nlm.nih.gov/articles/PMC6829626/)and [that!](https://pmc.ncbi.nlm.nih.gov/articles/PMC7121836/)
2) **Understand and treat diseases** along with predicting disease-causing mutations, as misfolded proteins are associated with a range of severe conditions *(particularly neurodegenerative like Alzheimer's, Parkinson's and Huntington's)* as well as some cancers. So by understanding how they misfold, we can design therapies to stabilize them or maybe enhance the cells natural quality control systems, or maybe even prevent the formation of the toxic aggregates that starts them all. Along with others mentioned in the [Disease prediction](#disease-prediction) section.
3) **Synthesize more proteins.** Maybe designing industrial enzymes that can break down plastic waste, creating more resilient crops, and many other.

**Summary:**
In AF1, we deal with a single protein, predicting how it will fold. Then, as the extension occur, we deal with complexes in AF2's multimer (along with single proteins in AF2 ver. 2.2). Lastly, we expand our capabilities of the two mentioned structures, all to view the wide range of biomolecules and predict their structures also. *Any relatively interesting detail will be added along the way.*

### The Problem Statement

The "protein folding problem" is actually three parts:

- **The folding code (Physics).** (Thermodynamic questioning). Basically all the explicit physical interaction rules that determine which folded state is "**stable**" for a given sequence.
- **Protein structure prediction.** The static, single structures [has been already determined](#alphafold-protein-structure-database-afdb)) (What we're doing in the [Engineering](#engineering) section). This provided the blueprint by having a static map of nearly all known proteins. 
- Understanding the **physical process and mechanism** by which a protein reaches its folded state in real time (*which is largely open kinetics*); including the intermediate states, pathways, chaperone assistance and **misfolding**. Meaning that we have a goal towards understanding "why" proteins fold the way they do. This question is abit out of scope, but we do address the misfolding and many other disease-causing mutations in a later section ([Disease prediction](#disease-prediction))

AlphaFold generally goes with (2) without having explicit inputs of the folding code (1) nor attempt to explain the models outputs (3). There are, however, some current case(s) of utilizing the explicit inputs of the folding code (used here in AlphaFold 1) indicated at [Outputs](#outputs), as well as later in AlphaFold 3s input. Regarding (3), this can possibly be examined in the model weights, but it is currently a black-box when relating it to AlphaFolds (complex architectures that weights aren't as easy to examine), that is, if we attempt to understand the algorithms output (probably a first-approach is finding a heat-map of activations just like how we did in CNNs and attention as indicated in the image below, but it is more complex the deeper the architecture is). 

We can also possibly build an XAI (Interpretable AI that its internal logic is understandable) to explain the currently **epistemically opaque** weights of the complex architectures. Either ways, it is definitely out of scope and may be included in future extensions.

![Pasted image 20251119130139.png](Pasted%20image%2020251119130139.png)

[Source](https://johfischer.com/2022/01/27/class-activation-maps/); Linear combination of the weights and feature maps to obtain the class activation map

**In other words;**
"AlphaFold" in general predicts the final structure, *but it does not fully explain the dynamic process of **how** it gets there in a biological system, nor does it address **the other [Formalisms and evaluation](#formalisms-and-evaluation),** which are mentioned below.* There are also an important conclusion: we may need [only the AminoAcid sequences as a "manual" in the folding process,](https://en.wikipedia.org/wiki/Protein_folding) (more on that [here](#building-up-from-the-smallest-to-the-biggest-protein-component)) Which comes from the mass-feeding of the data to the target model (in which it earns the intuition and pattern-recognition from thousands of samples). This proved to work (in the case of OpenAIs GPT-3 and many other LLMS) when scaling complexity and data, till [reaching a certain plateau](https://artificialcorner.com/p/gpt-5-may-be-proof-that-scaling-alone) that requires a redirection to innovating newer architectures or maybe re-framing the current use of what we have. More on that at the [AlphaFold Protein Structure Database (AFDB)](#alphafold-protein-structure-database-afdb). In the later implementation section(s), this is highlighted as "information bottleneck".
### Cancer

WIP
### Stem cells

WIP

*Small note*

The use of crystallography and OTHER techniques in identifying protein structure. They're an accurate way of figuring out the structure of proteins, but have you ever thought of other techniques, like Nuclear Magnetic Resonance (NMR), spectroscopy, Cryo-EM)? Well, if you go even deeper in that branch, there is a potential of a Machine Learning application towards traditional techniques like crystallography and these other techniques. So a Machine Learning algorithm to be used as an "Aid'er" or more professionally, a "Helper algorithm" in these techniques themselves. So besides a ML framework taking the core idea and digest (AF1's case), we'd have it as a secondary component that aids in the function of the core algorithm (or experiment if we're specific).

*In the next formalisms section, there is a "realization" of using some of these experimental techniques in solving the current frontiers of problems revolving around the protein.*

It also takes some form of delusion, too (mind you), as this is useful as a possibility opener if one is truly passionate about proteins and can open DOORS one couldn't have IMAGINED!! (yes, yes, bla bla) as long as it doesn't distract one from pursuing a single technique to their subjective creative limits, or however your working philosophy may apply.
### Formalisms and evaluation

*Some highlights in bold are of relative significance*

So, we conclude that:

1) [Proteins are dynamic, not static](https://pmc.ncbi.nlm.nih.gov/articles/PMC11623436/): they're constantly moving, twisting, changing shape in a biological environment. So when they perform their functions, say, binding with other molecules for example, any "Alphafold" predicts one molecular photo, similar to a static shot of a protein folded. But since proteins are dynamic and aren't yet captured by any "AlphaFold" model in particular, mentioning some more of the dynamics:
   - Transition states: The specific intermediate shapes it temporarily adopts moving between stable states (example being its interpolation between "open" and "closed" form).
   - **IDRs**: Some parts of proteins are naturally and inherently unstructured and flexible till they bind with their partners. This is actually crucial to interact with a wide range of client proteins. 
   - Real-time atomic fluctuations: Constant and rapid vibrations of chemical  bonds and residues.

Some experimental techniques (the one finding the structure of the protein itself) which are **NMR** and **spectroscopy** and **FRET**, along with molecular dynamics simulations that **do DESCRIBE** the conformational changes and the continuous flexibility and movement of the proteins in their natural environments (data we can use in machine learning). All of which would be key in figuring out not just one possible conformation, but all of their potential conformations per protein (one protein or a single "polymer") or complex (multiple proteins... the computation outlook seems large). Their methodology revolves around **stacking multiple protein structures (ensemble)** and transitioning between them from rapid atomic vibrations to slower domain rearrangements...

>... Which is somewhat related to Markov's formalisms of "Partially Observable Markov Decision Processes" in [Reinforcement learning](#reinforcement-learning), where we also (generally) stack frames when this problem arise (again, for later! And this is not the only method!)

Back at the experimental methods, there is a foreseeable future of datasets containing these just like PDB and such. Well, there is apparently a [method in combining multiple datasets](https://pmc.ncbi.nlm.nih.gov/articles/PMC3096476/), as well as developing NMR-only datasets like the one mentioned [here](https://pmc.ncbi.nlm.nih.gov/articles/PMC10767026/). The only thing we can do is wait for more NMR dataset(s) instances... unless we find out a faster technique or use what we have now (including the NMR or not).

It is also worth noting that the [protein folding process isn't a one-time event.](https://pmc.ncbi.nlm.nih.gov/articles/PMC9815721/#:~:text=While%20the%20importance%20of%20protein%20structure%20cannot,a%20typical%20protein%20adopts%20multiple%20stable%20conformations.). It is a dynamic and often reversible process. Some proteins need the help of other proteins to fold (chaperones), some unfold (denure) physiologically (like the natural process of protein refolding in muscles to allow muscles to stretch and recoil), others unfold to certain conditions. Amazing!!

2) The functional and interactional formalism: which would describe the proteins activity within the broader cellular context, interacting with other molecules and their surroundings (Protein-Protein interactions, Ligands, DNA: Which all seem to be partially or fully the target of AlphaFold multimer and Alphafold 3. They're still not following the first formalism, though) to preform specific biological functions. It would involve:
- **Network dynamics:** One a proteins action influences the behavior of an entire cellular pathway or network of interacting molecules.
- Kinetic properties: The rate and speed of biological reactions and interactions (enzymes kinetics, etc)
- **Allosteric regulation:** The binding at one side on a protein affects the function at a different, distant site by changing the proteins shape along with its dynamics.
- Environmental effects like solution conditions (pH, temp, salt conc) and presence of other molecules (ligands, other proteins) on the overall behavior of the protein.

There are also point mutations, which implies a single AminoAcid mutation that caused a structural defect. Similar to a pawn being promoted to a queen in chess that wrecks havoc on the advantage bar [which may be related to Reinforcement learning](#reinforcement-learning), too. This is also similar to the Network dynamics and allosteric regulation. Point mutations are the trigger of an ensemble of cascades of events. These are the structural basis for diseases, more on that at the [Disease prediction](#disease-prediction) section.

> It is advised to re-read this section after we're done with the small biology lesson.
## A Small Biology Lesson:

Proteins are made from instructions made by the DNA, and the RNA (the messenger of DNA). It, the protein, then folds to a 3D structure to do a specific function in the body. The structure itself is destined by *a mostly deterministic process* (i.e. native conformation). 

> More on that at [PDB bias](#pdb-bias) section.

![0KFB68iZ7qftEwSqI-768x447.png](0KFB68iZ7qftEwSqI-768x447.png)
[Source](https://en.wikipedia.org/wiki/Homology_modeling); an image of the DHRS7B protein (dehydrogenase/reductase 7B) created using Ribbon diagrams and rendered with PyMOL. This is not of concern now!

The AminoAcid sequence is the linear order of amino acids in a protein chain (referred to as the backbone or the "main chain"), meaning that we're currently hitting the main component of the protein structure. Also are the [1. Primary Structure](#1-primary-structure)

AminoAcids (AA's) are made up of a carbon atom with a carboxyl (COOH, acidic) and an amine group (NH2, written H2N which is a base) each to the center (the carbon). The north bond being the Hydrogen on the north, and the last bond (south of the AA, called R group) would be either of the side chains.
Please don't mind the pole directories I mentioned.

![Pasted image 20251110110650.png](Pasted%20image%2020251110110650.png)
![Pasted image 20251110110830.png](Pasted%20image%2020251110110830.png)
![Pasted image 20251110111132.png](Pasted%20image%2020251110111132.png)
[Source](https://www.youtube.com/watch?v=P_fHJIYENdI): Veritasium; attempting multiple different side chains, which are later binded to other AAs forming a linear chain (that is different from a side chain). That straight line represents a linear chain of the carbon (center of the amino acids!), each bound to a side chain [till they form a protein or become part of a complex?]

This random choice between different side chains ends up forming the 20 AA's that exist in nature, not taking into account the unknowns, and the rare AA's. They're encoded in Alphabetical names, such as "`U`" for the Selenocysteine AminoAcid (there are some not encoded with their respective starting word). The unknown one(s), or harder to distinguish between other AminoAcids are as following:

1) `B` (Difficult; may be a `D` or `N`),

2) `Z` (Difficult; may be `E` or `Q`),

3) `J` (Difficult; may be `L` or `I`) and finally:

4) `X`(Undetermined or any unknown AA. Very interesting).

Along with the rare AA's, which are the AA's no. 21 and 22 respectively:

1) `U` and 

2) `O`

Some of these are not needed in human trials, for example the `O`, found in certain archaea and bacteria unless there are specific needs or research to be conducted on these organisms.

You can call the 22 AminoAcids "Proteinogenic" AminoAcids, which implies that they are used for building the protein from the sequencing and folding process. It's important to note that there are over 500 naturally occurring AminoAcids that have been identified, in which case are called "Non-proteinogenic" AminoAcids. They are beyond the scope of proteins and the alphabetical order, as in they do not serve the function as to become proteins, but they serve other biological functions. The potential of creating [newer AminoAcids](https://www.nature.com/articles/nchembio847), however, are being researched by modifying the R group (the "legally" modifiable part). Unless you modify the core parts (COOH and NH2), in which the AA may not be recognized by the cells machinery


![Sequential, Structural and Function representations, as from our previous blogpost. 1.webp](Sequential%2C%20Structural%20and%20Function%20representations%2C%20as%20from%20our%20previous%20blogpost.%201.webp)
[Source:](https://www.ml6.eu/en/blog/esm3-the-frontier-of-protein-design) Sequence determines structure then structure determines function --- and .. Oh, wait:

![Pasted image 20251125121847.png](Pasted%20image%2020251125121847.png)
[Source](https://pmc.ncbi.nlm.nih.gov/articles/PMC12467925/): An amazing article about the De novo protein design of RF diffusion. They also have an amazing table on the protein-design workflow!

In this current decade, we can and are able to "create" proteins for specific functions that we dictate for a model to create a 3D protein (extremely cool I know!). They use the Proteinogenic 20 (sometimes the other two) for this, not by creating new AA's. The process is very similar to creating images using AI, Briefly; having noise at the beginning, then iteratively removing the noise to generate a new protein structure that is a "plausibly folded structure", maybe even incorporating a novel proteinogenic AminoAcid (engineered) in the diffusion process (or other parameters like symmetry specifications and length ranges). 

>Keep in mind that this is a generated backbone, and we still don't know its sequence of AAs.

Small note: It is very similar to predicting the next word in a sentence, where we iteratively remove the noise, and the model starts seeing patterns linking the text of description we gave (tokens) to its embedding space tokens. Its also similar to seeing patterns out of a seemingly redundant static noise. Nonetheless, It is deemed interesting but slightly out of scope.

![Screen-Shot-2023-01-10-at-3.26.51-PM-750x366.png.webp](Screen-Shot-2023-01-10-at-3.26.51-PM-750x366.png.webp)
[Source:](https://zontal.io/rfdiffusion-leveraging-the-power-of-ddpms-to-generate-protein-sequences-and-structures-2/) The amazing Rf diffusion!

There are more nuances to that, such as using [ProteinMPNN](https://github.com/dauparas/ProteinMPNN) to design a sequence of AAs that is most likely to adopt that specific 3D shape, then use AF2 or AF3 to verify the structure viability (sanity checking if it will fold back into the designed structure) before costly lab synthesis begins, that is, if we're attempting to create the protein.

> Psst! This model is a fine-tuned version of the RoseTTAFold structure prediction network (similar to AlphaFold)! Check out [BakersLab](https://www.bakerlab.org/2023/07/11/diffusion-model-for-protein-design/)!!

[More on that is a potential for integration in this article]
### Building up from the smallest to the biggest protein component 

The levels of protein structure are as follows:
#### 1. Primary Structure

These are the most fundamental level of "structure", which refers to the linear sequence of AminoAcid themselves. The covalent bonds (Called "peptide bonds" here, linking an AA with the other... sometimes disulfide) are keeping them intact (more on that below). Contrary (Opposite) to the interactions other than that, which are to be considered "absent" within the primary structures stability and integrity under stress. 

![Pasted image 20251129112651.png](Pasted%20image%2020251129112651.png)
[Source](https://www.genome.gov/genetics-glossary/Amino-Acids); AminoAcid chain

There are also conformational freedom, which are done by the partial double-bond, which are also called "trans-conformation" of the peptide bond between two AA's... and its strength is somewhat between single and fully double bonds. What matters here is that the angle is nearly always fixed at 180 between only two AA's, conveying a planar structure. So the bond has the proteins on a "strong approximation" set of an absolute physical constant angle called 𝜔 (omega). 
#### 2. Secondary Structure:

These are **alpha helices** and **beta sheets**, which are defined by specific, repeating patterns of . The secondary structure of a protein is related to a type of feature-in-interest (in the note below) more than the primary structure.

>The Carbon (C𝛼) backbone are bonded with NH2 (single bonds, flexible) and have an angle, **which are called Phi ϕ torsion angles**, while the Carbon (C𝛼) and COOH bond angles are **called Psi 𝜓 torsion angles.**

SS is defined by specific, repeating patterns of the PHI/PSI angles along the backbone of the polypeptide chain. The values of these angles determine the overall conformation of the backbone in a specific region, thereby, it dictates whether that region forms a helix, a sheet, or a random coil.

![Pasted image 20251129125947.png](Pasted%20image%2020251129125947.png)
[Source](https://chem.libretexts.org/Bookshelves/Organic_Chemistry/Organic_Chemistry_III_%28Morsch_et_al.%29/26%3A_Amino_Acids_Peptides_and_Proteins/26.10%3A_Protein_Structure): The main chain (called the Cα coordinates) as well as the side chains

The "Ramachandran plots" are plots defining the *sterically* allowed conformations of the PSI/PHI angles so the atoms wont go crashing on each other.

![Pasted image 20251129132710.png](Pasted%20image%2020251129132710.png)
#### 3. Tertiary structure

Since the tertiary structure is the overall, complex 3D shape of a single polypeptide chain (and the one AF1 attempted to "solve"), which includes all the secondary structure elements and how they fold; **there is a conclusion that the proteins tertiary structure (the 3D fold) is determined by the interactions between the AA side chains (R group) and between other side chains with other!** Their interactions (more on interactions below) cause the stabilization of the 3D shape. There is also an influence between side chains and the atoms of the peptide backbone.

#### Domains in proteins

WIP

#### 4. Quaternary Structure

For the last structure, there is Quaternary protein structure (complexes), which has proteins consisting of the (1), (2), (3) structures, called now "subunits" (two or more proteins in the entire complex) that we mentioned.


**Summary:**
![71225d815cafcc09102504abdf4e10927283be98 1.png](71225d815cafcc09102504abdf4e10927283be98%201.png)
[Source:](https://www.khanacademy.org/science/biology/macromolecules/proteins-and-amino-acids/a/orders-of-protein-structure) Honestly an amazing demonstration for the bigger picture view, maybe have some terminologies in brackets like pleated sheets should be "Beta Pleated sheets" but that's only to aid beginners in learning structure terms).
#### The predictors of the tertiary structure

In our static protein structure prediction problem, we can safely say that the strongest predictors are their primary structure, **the sequences**. It is a deterministic way of finding out the final 3D structures, although even sometimes some proteins have different AA's and still incur some attributes that of a different AA sequence (see [2 MSAs homologs](#2-msas-homologs)). Their properties also matter:

1) The order of AA's: Which is the primary structure predictor. This contains all the information needed for the protein to spontaneously fond into its correct, functional 3D shape.

2) The types of AAs

3) The third property is the No. of AA's, which defines the overall length of the chain. This is relevant because it defines the size and complexity of the resulting structure. The longer the chain the more opportunities it has for folding.

There are "attraction forces" coming from salt bridges, we need positive and negative charges together so the entire protein can be stable. As long as there is charge balance, everything can become stable.

These two properties together (types of AA and their order) are crucial as they dictate which interactions can occur. As stated on [The Problem Statement](#the-problem-statement), the folding code (1) are the explicit rules of folding **in one of the two interactions**; the predominant ***non-covalent interaction(s)***: hydrophobic effects, H-bonds, van der Waals forces and salt (ionic bonds) bridges. They solve the core question of "Why is this folded state energetically favored, and not the other states?"

![Pasted image 20260130191838.png](Pasted%20image%2020260130191838.png)
[Source](https://commons.wikimedia.org/wiki/File:Accessible_surface.svg#/media/File:Accessible_surface.svg): Van der Waals, as well as Solvent Accessible Surface Area (more on SASA/ASA at the [B. Auxiliary/Intermediate Loss Functions](#b-auxiliaryintermediate-loss-functions))

They are weak individually but act together (thousands of them!) to stabilize the protein. Their drawbacks is that how easily they're disrupted from heat or changes in ph. We also assume no extraordinary environmental conditions as mentioned in [PDB bias](#pdb-bias).

The other type of interaction is the ***covalent interaction***: disulfide bridges (or bonds). These bonds are stronger than the former mentioned non-covalent bonds. There is already a foreseeable potential of its aid in the [Disease prediction](#disease-prediction) section, maybe also a previous section that you can connect to.

These interactions as a whole put us on the importance of the [2) MSA](#2-msa). Which has one sequence far away from the other (in the same primary structure chain, speaking of a single protein) that are also touching [Important finding!]

>The interactions were *not* explicitly fed to the algorithm as it implicitly learned them. Maybe that's for the best, as sometimes exploration ([Reinforcement learning](#reinforcement-learning)) is more important than being given foundational knowledge. It is learned implicitly through the scaling of the network and data.

One other predictor of protein structure are Multiple Sequence Alignments [2. MSAs homologs](#2-msas-homologs) which are explained later and is the main signal used in AlphaFold 1.

Keep in mind that we have protein (or related) "lessons" also scattered around the document, as some information will be pulled when required.

Now, let us us demonstrate the popular dataset at hand:

## PDB

The central Protein Data Bank has explicit [entries](https://www.ebi.ac.uk/training/online/courses/exploring-pdb-entry/) of:

1) **The 3D atomic coordinates** (fundamental target data for later) which allows the visualization of the final static snapshot of the folded protein. This is a very precise X, Y and Z location of every non-hydrogen atom (**sometimes** ( * ) hydrogen) in the resolved structure.

![Pasted image 20260206141528.png](Pasted%20image%2020260206141528.png)
[Source:]() Fun fact, this data structure in PDB has changed in ID's for 5 years consecutively. I believe from 4 to 5 to 12, but it's a great reminder that datasets do change.

2) Bound Ligands/Molecules which are coordinates of water, ions (zinc, magnesium), drug molecules that are bound in the structure. They are non-proteins that were co-crystallized or co-solved with the proteins. May be useful in the deeper aspect of [The Problem Statement](#the-problem-statement)

![Pasted image 20260206141843.png](Pasted%20image%2020260206141843.png)
[Source:](https://www.researchgate.net/figure/a-Scheme-of-the-ligand-binding-of-a-small-molecule-to-the-target-proteins-b-Binding_fig1_285783261) Amazing big picture image illustrating ligands and molecules interactions

3) Multiple conformations (limited): NMR ensembles are models, meaning a small set of 10-20 representative coordinate sets
4) Experimental metadata:
- Experimentation methodology: X-ray crystallography (traditional, accounting for ~87% of all the PDB archive), NMR (, Cryo-em (Cryo is advancing rapidly!), which are the big three used in PDB. There are still other techniques used like SAXS, CD and other. Maybe also the **experimental conditions** that are crucial as indicated in the [Formalisms and evaluation](#formalisms-and-evaluation) section
- The resolution or quality of the data, where higher resolution indicate a better "zooming in" the protein to identify the structure and its fine details. The higher the (Ang.), the lower resolution that we get. Similar to a pixelated photo, you can see the backbone, but not the individual parts like atoms and side chains and the opposite applies here.
- R-factor and R-free, B-factors, clash score, RSR and other. Check out [the amazing documentation BY PDB assessing the quality linked here](https://www.rcsb.org/docs/general-help/assessing-the-quality-of-3d-structures#:~:text=mismatches%20between%20the%20model%20and%20the%20experimental,to%20errors%20in%20model%20building%20and/or%20refinement.)
 - Source organism, and other data like sample preparation and refinement statistics.

> ( * ) Hydrogen being "sometimes" in the data is due to its weak signal and experimental limitations. They can be also too predictable, making it "wiser" to omit them (increasing computational efficiency) even though they are the crucial for protein folding and function.

The potential of application (of the quality and metadata) relies on their ability to be ingested to the Neural Network (NN) and to reduce their weight, **as the less accurate the experimentally decided structure is the lower value we have on its potency to the predictions outputted.** Surface side chains, disordered regions and flexible loops are other examples on weak or no experimental data due to their inherent motion or disorder in the experiment. They can be useful, however, if we were trying to innovate something here...

*Some of which should be derived.*

Remember the second formalism of proteins, the one about environments?
We have data from the infamous [ProtienDataBank](https://www.rcsb.org/#Category-deposit) (PDB), in which  the experiments in it (commonly) gotten from the X-ray crystallography has an environment **which probably misses some important details:**

 - Non-physiological PH or salt conc. (These were used to force crystallization)
 - Low temperatures (almost always, unless specialized experiments. Room temperature is increasingly becoming the norm here)
 - Absence of binding partners or cofactors that normally are present
 - Dehydration, etc

### PDB bias

*This follows the aforementioned [Formalisms and evaluation](#formalisms-and-evaluation). It also uses "AF1" (our target understanding algorithm) to demonstrate the use of PDB. Targeting the missed second formalism as well as other notions of an "accurate protein structure prediction"

Protein folding, as a problem statement in itself is both a deterministic (somewhat NOT random) and stochastic process (somewhat random). Its determinism comes from the AA's specific sequence, as ***most*** proteins with the exact same AA sequence have obviously the same shapes (scientifically, the "lowest-energy state"), while the environmental factors and random thermal motions (pH, ions, ligands, etc) add the stochastic elements in the folding process. 

***AF1 uses the deterministic side of the equation.*** In other words, they assume stable environmental and specific laboratory conditions. At the worst case scenario, there there will be misfolding, and all the bad things could happen if a protein is introduced to an extreme environment.

At every other scenario, even a fever of `39c` or so, there is little to no effect to the protein and stochasticity which implies a more deterministic outlook that we have currently have (sufficient average determinism). Only after that point, though, does it compromise proteins and its folding process, and the chances increase from thereon.

The uncertainty in AF1 predictions, where even a `1-2A` difference is considered uncertain, is much larger than a couple Celsius shift in temperature. Though marginal activity may drop but still has no significant effect, maybe even more "flexible" and within biological tolerance (The effects that I'm currently aware of. DO NOT solidify that as a foundational barrier to your brain).

Therefore, unless we're making protein fold in "possible" scenarios, **this may not matter as much.** This might affect our abilities in understanding and solving some diseases [Disease prediction](#disease-prediction), it's great to keep this in mind, though, as maybe there would exist a fever model or a very mythical application in which I do not endorse, maybe a certain environment-based protein folding process in the future that adds on to that layer, or something according to your liking.

**And also to note** that there are other databases (DB) that were used by AF1, such as UniProt (big brain), Uniclust (targeted brain), BigFantasticDatabase (BFD) and MGnify. Some of which DBs are related to the protein itself and its sequence and such, and the other may be related to the [2) MSA](#2) MSA). These were skimmed as we'll give them their fair chance later on in the engineering part.
#### AlphaFold Protein Structure Database (AFDB):

Now we know about the Protein Data Bank (PDB) and many other available Databases to tackle the protein folding problem, it's great to know that the predictions utilizing the PDB database (the one we're building for example) have their own database of predictions. There are over 200 million protein "highly accurate" structure prediction from the AF2 algorithm. We have the confidence scores of the fold predicted compared to the structure from the PDB called per-residue confidence score (pLDDT, which should be over ≥ 90 to be considered "Very high confidence").

![molecules-28-07462-g006-550.jpg](molecules-28-07462-g006-550.jpg)
[Source;](https://www.mdpi.com/1420-3049/28/22/7462) Red shows high confidence areas and blue indicating a lower confidence score. They were used as primary losses in AF2.

The lower we go, the more likely we reach a "flexible" protein zone of prediction, meaning that the algorithm cannot predict it well (or it doesn't have the information that's "satisfactory" level to model these, or maybe they're just random regions or coils "IDRs"). Nevertheless, it's great to mention AFDB.

![science.adq4946-fa.jpg](science.adq4946-fa.jpg)[Source](https://www.science.org/doi/10.1126/science.adq4946): One of the many uses of AFDB

It is also to be noted that reliance on predictions to make predictions (or in other words relying on theories to make a theory) isn't in essence a bad thing,
as maybe theory 1 may be right (proved or assumed from the conclusions it produces). which makes all the other theories it produces plausible. Just how the quantum is field is 

### Reinforcement learning

![Pasted image 20251223213252.png](Pasted%20image%2020251223213252.png)
Source: Reinforcement Learning by Maxim Lapan (Cool book)

How can you apply reinforcement learning here? The insight was mainly formed because reinforcement learning (RL) has both *stochasticity* (at the beginning with possible environmental changes), and also has *deterministic* phases in later stages (by decreasing the epsilon-greedy parameter) in some foundational algorithms. But that's only intuition, and not truly touching on the main idea of the document! Like increasing stochasticity the more temperature we rise in the folding process, and subsequently decrease the determinism factor in it! That's a similarity remark in processes, which doesn't provide actual "insights" unless

You can start small, because the reward signal maybe is the challenging part. As some are sparse and some are goal-based rewards (which require knowing the answer), 

But if we think about it as a non-originator algorithm (not being the core algorithm), RL may shine in refining coarse predictions from AF1 itself, that part that AF1 missed which is the stochastic parts, in which we have:

- **State:** Current structure from AlphaFold, as well as the MSA and probably every data ingested at the Next section
- **Action:** Local perturbation (adjust a region) 
- **Reward:** Energy decrease + structural validity
- **Policy:** Learn to make smart local adjustments that respect physics!

Maybe if we even incorporate video output to the pipeline, for the algorithms to have a chance in modeling the complexities and incorporate a game just similar to [Foldit](https://en.wikipedia.org/wiki/Foldit) (molecular dynamics intact and have a state), then, we can probably think about their practical extension and application, along with adding players back into the field... or any other field, really.

# Implementation and Intuition:
## Bigger Picture

There is an input, and there is an output (Stage 1). Then, there is a Structural Construction stage, where we utilize physics-based constraints on all the inputs from stage 1 to get the final 3D coordinates. Then, optionally, we attempt to reduce the energy to its lowest possible energy state (of course, based on what the model can handle with its current complexity!) using also a physics-based energy minimization. This third stage (optional) uses the 2nd stage as input and has an output of a refined 3D coordinates implying a yay!-level-lower-energy state of the protein predicted.

Since ***most*** architectures aren't using the "physics" aspect as modules anymore, as that they avoid explicitly encoding or using the physics alltogther by gaining enough PDB examples, it's somewhat plausible that there is a way to skip some parts working by using physics to learn the lowest-energy state (the folding process) and go pure pattern recognition from MSA's and structural data. This is a possibility to note. (Nevermind, this is AlphaFold2-RAVE and others! and nothing is invented here)

> Physics modules are still essential! Ones for the drug binding that respect the physics aspect, like AutoDock, and others are for protein dynamics (data collection), and some other for validation (AMBER force-field-min) because machine learning in general *can* hallucinate structures (in AF2 case) or provide inaccurate constraints from the neural network (or general clashes).
> 
> It's always useful to see its applications even if it is "outdated". Most likely, both components can be paired in a way.

AF1 didn't represent the minimum complexity of modeling proteins, it achieved a rather significant advancement than the simple **template-based modeling** (same protein == same structure) and Ab initio *which uses physics energy functions to simulate folding* (in which also rarely worked well, but hey, it may potentially have a comeback). Basically, you can say AF1 is the proof-of-concept that one can apply DL techniques on proteins, with other architectures using the same dataset of PDB, also using different methods; such as eliminating the need of MSA and such. Which is task dependent, obviously.

*Maybe all it needed was more essentials and lesser complexity.*
## Training


![Figure 2 1.jpg](Figure%202%201.jpg)
[Source](https://www.nature.com/articles/s41586-019-1923-7.epdf?author_access_token=Z_KaZKDqtKzbE7Wd5HtwI9RgN0jAjWel9jnR3ZoTv0MCcgAwHMgRx9mvLjNQdB2TlQQaa7l420UCtGo8vYQ39gg8lFWR9mAZtvsN_1PrccXfIbc6e-tGSgazNL_XdtQzn1PHfy21qdcxV7Pw-k3htw%3D%3D): The nature paper, this image should make sense by the end of this section

The main data at hand;
### 1. AA sequence

Three letter words; `ALA` for Alanine, `CYS` for Cysteine, `ASP` for Aspartic Acid, etc) that we sometimes have to convert to their standard one-letter codes that we mentioned in [A Small Biology Lesson](#a-small-biology-lesson).

"`MKTAYIAKQRPGLV`" an example. They can be subunits as in Quaternary  structures (hemoglobin, which has 4 subunits of these sequences) or a simple sequence of a single string of protein AA's. These are not used raw, and is used to search for MSA (next section).
### 2. MSAs homologs

![Pasted image 20260208230432.png](Pasted%20image%2020260208230432.png)
[Source; Veritasium](https://www.youtube.com/watch?v=P_fHJIYENdI); The Multiple Sequence Alignment

In simple terms, we see if two AA stay conserved across different specifies and other AA that may have co-evolved. Meaning that they mutate together, which often signals that they're often close in physical contact when folded to 3D structure. This is crucial input to the NN.

**Take a single protein for example**, we'll call it hemoglobin as a scientific name over "blood" (Bear with me for the simplification):

"`MKTAYIAKQRPGLV`"

Here, we say that "`M`" and "`T`" are pretty close in that 1D sequence (the string of characters you see here), it may be temping to say they are close to each other in 3D. but proteins can surprise you with how far they actually are. For this specific case, these are called structural significance. As in, `M` and `T` maybe are not close in 3D space, but they are important for the structure itself. Don't mix the two ideas.

Now, let us introduce the meaning of "MSA" and how useful can it be by its amazing two aiding intuition: Co-evolution and General Conservation.

*Imagine the same protein in 1,000 different species:* 

```
Human: MKTAYIAKQRPGLV... 
Chimp: MKTAYIAKQRPGLV... (99% identical) 
Mouse: MKTAYISKQRPGLV... (changed position 7: A→S) 
Fish: MKTVYISKERPGLV... (changed positions 4,7,9) 
Bacteria: MKTVFISQERPKLV... (many changes) 

Pattern: some other organisms have similar AA sequences to a protein in question
```

Say that it was concluded that positions 7 and 150 are in contact in 3D (We x'rayed it); If position 7 changed from AminoAcid `A` to  AminoAcid `S`, THEN consequently, position 150 (in which it is too far to be written sequentially here) also changes to maintain interaction (Which is called: Co-Evolution!) This is the point of MSA and homologs in general, which is to find this interaction and conserved regions by arranging related protein sequences).
##### Co-evolution vs general conservation [wip]

and we have "General Conservation", which is the simplest signal reflecting the structural importance of an AA position. Meaning that highly conserved sites are critical for

So we're basically saying here: "If two AA positions in a protein are close (in the final structure), they likely have co-evolved over millions of years."

[What's the correlation between co-evolution and contact in the 3D structure?! TO DO]
##### How "Identical" should we aim for?

As with this scope, it may seem vague as we would be optimally utilizing any form of similarity between homologs toward predicting structures. In the case with AF1, they utilized both techniques as to utilize even the weaker links between homologs (see [Remote or distant homologs](#remote-or-distant-homologs) and [Orphan proteins](#orphan-proteins)) either ways, **they shouldn't be a constraint** if we're looking for that along some smart engineering designs.

30-50%< match between the AA in question and its homolog(s), as a rule of thumb towards its viability, as to use in template-based matching till a 80% match. Else, they'll be considered "[Remote or distant homologs](#remote-or-distant-homologs)", even worse for the case of the reliant AF1, one of the [Orphan proteins](#orphan-proteins).
#### Remote or distant homologs

There are certain proteins that look similar to certain others in their 3D structure, while oftentimes having also similar functions, all due to having descended from a common ancestor (being evolutionary related). But at the same time, they have a VERY LOW AminoAcid sequence-sequence similarity, maybe even undetectable because of the huge evolutionary time. The "metric" here is close to a <30% match and below till 20% between the given AA sequence and the homolog.

It's important to note that, despite their vast differences in the AA sequence, **there can be a similar structure AND function between the homolog and the AA sequence!** Because that fold works, even if the AA seq are different. Nature says; "if it ain't broke, don't fix it".

Here, we can be skeptical, though. As there will be structural details different (obviously, as the AA sequence is the primary driver of its structure as we previously mentioned [1. Primary Structure](#1-primary-structure))

>The use cases may be that if we had a new protein sequence without knowing their functions, we could search for a possible remote homolog to infer the functions.

Therefore, we can conclude that we have two input features;
the [1. AA sequence](#1-aa-sequence), and 
the [2. MSAs homologs](#2-msas-homologs)
#### Orphan proteins

These are proteins without homologs (No MSA depth, in other words, has no neighbors; similar to the **Tetherin** in vertebrates). Here, the sequence itself becomes the only input available for the NN, alongside the meaning that AF1 gets zero co-evolutionary information. We model the given AA sequences using "Free-modeling" (FM) in this case, not even a mix between it and TBM

There are other algorithms utilizing language models like "RGN2" and "ESMFOLD" and "trRosettaX-Single" to predict structures. One may think of combining the two, or utilizing the other when there are no homologs available, the other **may think of generating Synthetic MSAs** (beneficial for Alphafold and its variants) like "(GhostFold)" and such. So anything out of MSAs is definitely out of AF1 (unless you're seeing something that I cannot see).
### Loss functions

Loss functions are the predicted value or representation and comparing it to the actual representation that a neural network or a machine learning algorithm should have predicted (Ground-truth labels). In other words, the difference (minus sign here) between the predicted value and its actual value. **The folding code (rules and interactions) are learned here.** The loss function guides the massive number of weights to learn the correct patterns, **meaning that the NN optimizes for structural accuracy,** implicitly learning the folding code that is mentioned at the [The Problem Statement](#the-problem-statement).

Adding onto that, at the [Second stage](#stage-two), there will be no "optimizing weights", just something that's similar to it in the wording, called "Energy minimization". So most of the following loss functions are used strictly in the NN process.

The loss function outlook is just as large as AF1 multi-task learning technique. One neural network is trained on multiple outputs, as well as the hope that the "shared structure" improves performance. Because, having a shared structure of 

The loss functions are split between two groups: 
[A. Primary Loss Functions](#a-primary-loss-functions): Distogram
[B. Auxiliary/Intermediate Loss Functions](#b-auxiliaryintermediate-loss-functions):  Torsion Angle, The Secondary Structure, SASA
#### A. Primary Loss Function

##### The True Distogram

Also called "**Inter-residue distances**"; They are the distances between two residues (A pair of AAs). Here, we compare the distance between each AA and another in the **same protein** at a 3D space. They're often categorized by bins as will be mentioned in [Engineering](#engineering). The Distograms aren't provided plainly from PDB, but they are DERIVED from the "Atomic Coordinates" (more on that at the [Final Output; from the previous stage](#final-output-from-the-previous-stage)).

![Pasted image 20260129224218.png](Pasted%20image%2020260129224218.png)
[Source](https://discussions.unity.com/t/use-case-distance-matrix-3d-protein-biology/308143); What the usual distogram looks like

*Example (distogram) illustration between five amino acids in the same protein:*

```
    M    S    V    T    Q 
M  0    3.8   7.2  12.1  15.3 
S  3.8   0    4.1   8.9  14.2 
V  7.2   4.1   0   5.3   10.1 
T  12.1  8.9   5.3   0    4.8 
Q  15.3  14.2  10.1 4.8   0
```

So they're distances between pairs of AminoAcids. You can see that the distance between AA "`M`" and itself is zero on the top left corner (similar to the distance between you and yourself), while other pairs facing the same pattern. There is also a pattern hinted by a diagonal line (as you've also seen in the image above)

*You can optionally switch them to Binary Contact Maps when hit a range:*

```
1 = if distance is below 8Ang.
0 = Above 8Ang.
```

*Example Illustration of Binary Contact Maps:*   

```
    M S V T Q   
M   0 1 0 0 0   
S   1 0 1 0 0  
V   0 1 0 1 0   
T   0 0 1 0 1   
Q   0 0 0 1 0
```

###### Motive of the distogram:

The idea on choosing the distogram is due to **its similar properties with the MSA:**

Compared to our other loss functions, this is only **global information,** meaning that it involves residue (AA) **pair interactions** (Contacts, folding topology, domain packing) far apart in sequence: Res20 with Res150 as explained in the previous MSA section. As for the **local information,** handled by mainly our PHI/PSI torison angles as well as the secondary structure.

In AlphaFold 1, researchers turned protein data into that distogram (a 2D grid showing the distance between every pair of amino acids). They treated this distance map _exactly like an image_ because they assumed that if amino acid A is near B, and B is near C, there’s a local "shape" to be found. Although this was a constraint, which was alleviated using AF2 (the Transformer architecture).

>*"What can the NN conclude though?!"*

- Some AminoAcid pairs are far apart in 1D, but are close in space. Then, this is possibly a folding interaction (which are exactly what the [2 MSAs homologs](#2-msas-homologs) encode)
- For other auxiliary losses, Close residues (`3-4 Å`)? Then a match in helix shapes, Variable distances then suspects a loop form (both are secondary structure)

The NN can conclude the characteristics of a given sequence and distance map and use the pattern it found to predict the distance maps and others, and it keeps iterating on that till it has finer predictions (with emphasis on bias and variance).

#### B. Auxiliary/Intermediate Loss Functions

They are the loss functions that aid learning early on (Scaffolding), as the global information is known from accumulative local information.
##### 1. True Torsion Angles, PHI/PSI
 
 It uses the backbone and side-chain dihedral angles. Basically the rotational **degrees of freedom** (the intuition) that define the SS (indirectly in practice) of Alpha and beta sheets! They were learned, and they provided knowledge on how "much" the backbone could physically bend and twist on the Ram-plots previously defined. Torsion Angles are compromised of **Phi** and **Psi**. (See [2. Secondary Structure](#2-secondary-structure))

![Dihedral-angles-in-glutamate-Dihedral-angles-are-the-main-degrees-of-freedom-for-the 2 1.jpeg](Dihedral-angles-in-glutamate-Dihedral-angles-are-the-main-degrees-of-freedom-for-the%202%201.jpeg)
Source: https://www.researchgate.net/figure/Dihedral-angles-in-glutamate-Dihedral-angles-are-the-main-degrees-of-freedom-for-the_fig4_44651362

##### 2. **True (S)econdary(S)tructure (SS) profile**

![Pasted image 20260130191143.png](Pasted%20image%2020260130191143.png)
[Source](https://www.researchgate.net/figure/D-structure-of-ligand-free-sterol-carrier-protein-2-like-2-from-Aedes-aegypti-Protein_fig6_326220346): The secondary structure of a protein (see the previous [2. Secondary Structure](#2-secondary-structure))

Although they have their own head as output (just as the other loss functions), they're often derived from the PHI and PSI ranges( * ). *These help learn the local geometry as stated before.* It outputs a probability of each position being either:
    - `H` (helix: spiral-looking)
    - `E` (strand-looking)
    - `C` (coil: strand but in an unstructured form)

##### 3. **True Solvent accessibility SASA/ASA**

The actual amount of surface area of each residue exposed to the surrounding solvent. Binding sites are likely to have High SASA, **but not vice versa**. Meaning that an exposed part of a protein may likely be a binding site for whatever the purpose of the protein in question is to do [Cut off]. This helps the NN learn about hydrophobic core formation, along with loop regions. **Scientists check if hydrophobic residues are properly "buried" away from the solvent as expected in a stable fold.** The SASA is found by the rolling-sphere algorithm

By adding "True" in these loss functions, we're emphasizing the current stage in question *(i.e: Training, these are given ground truth derived data, as they won't be available in inference).* Adding onto that, the two loss functions SASA and SS will not be used in the later stages, but it is useful to attempt and find some form of use for them downstream.

These auxiliary losses aid the NN to learn, **but remember, we don't specifically use them in other than the neural network to aid learning. An exception is the PSI and PHI angles that help in later stages.*** For the auxiliary losses, use these as additional training signal during training with a very marginal weight provided *(not affecting the outputs as much as other loss functions like Distance Matrices and the Angle Matrices, as they're with a priority here).*
### Outputs:

> Note: AlphaFold 1 produced several outputs that described the proteins conformation in a 2D, pairwise representation which are essentially structural features and local geometry information.

#### Stage one: NN's four heads outputs four variables

...they're also probability distributions, ***not single values,*** using the name of all the four aforementioned loss functions; Torsion angles, SS, SASA, and the Distogram.

*The loss functions are NOT the same as "output heads", keep that in mind.* 

Here are "Shared latent representation" of a single NN type *(A Conv-net, which are used not only for images as some practitioners assume. More on that later)*, and the probability distributions are the outputs of the four heads from that shared latent space of the neural network (Emphasizing efficiency).

>AF1 is not predicting structure.  
  It is predicting ***constraints***, using evolution as a sensor. The final stage is geometry constructed using the provided constraints.

**The network, basically, is heading toward the distogram, as AF1 authors (as stated before), put more weight on it than any other head.*** All the other heads are "regularizes" that shape the loss landscape and make learning easier early on for that distogram head (priority). In practice, **you can add more heads as desired. E.x;** contacts, hydrogen bonds, disorder, interface probabilities. But, if these novel heads won't add a new learning signal *(other than the four we currently have)*, then they won't improve our target: The distogram. Worse yet, they will make training performance degrade (information bottleneck theory).

The paper authors tested *ablations* (what if we removed component X? Any changes in accuracy? Any changes in component Y's behavior?), and by removing the distogram the model had collapsed (a head of significance), whilst other components shown lesser performance drops.

>( * ) Note: SS are inferred/derived from the PHI and PSI angles themselves using other algorithms like DSSP logic to assign H/E/C by analyzing angles of the distance maps all in all after the NN prediction. If we wanted to know and predict the SS before the model outputs a 3D structure, we use the torsion angles and ingest it to a "PSIPRED" or a "S4PRED" to get a prediction. Whichever fits the speed/accuracy goal respectively. 

**A Question I had, now answered:**
*Information content separation from training signal:*

**In theory:** SS can be **"inferred/derived"** from φ/ψ (PHI/PSI torsion angles that were outputted), so it's a ground truth label from the predictions themselves, which fits the provided PHI/PSI that the model predicted. So why do we even predict the SS in a different head? Isn't that computational overhead?

The issue with "inferring", for multiple reasons;

1) Is that "inferring" itself doesn't have **gradients** (i.e: the neural network doesn't "learn" from inferring). The Auxiliary heads we have are not about correctness, but they are about ***conditioning the geometry of the representation space*** (Machine Learning). 
- So "infer-able" means to be able to compute something after the fact (output)
- As for "supervisable", it is able to provide gradients during training, which shapes learning

Using ML terms, in the case of the "inferring" route, the NN algorithm can reach the same solution, but SGD **has no reason to walk there efficiently** 
(*Stochastic Gradient Descent, it is an optimizer of any loss function. More on that are later at the engineering section)*.

2) Again, the head(s) output a probability distribution of the PHI/PSI angles. Would it be wise and derive all of the probable values that we get? Well that means we can add as much outputs as we need, but there has to be a loss that guides it in a way that it knows that it does its thing (prediction) correctly; hence the "Supervised learning" term comes through.

 >"What if we switch to a single value output only for the PHI/PSI head?"

Well, to infer the SS, we still need to compute the X Y Z coordinates and such which is the output found and gotten from the second stage (next stage).

3) The real computational overhead is comparable and actually, is more from the act of inferring itself, not from the "extra" outputs. 

> in (1), It's almost similar to Reinforcement Learning when it is trained to play chess and we hand out the game rules to make training faster and less noisy at the start of training, avoiding the "reinvention of the wheel". However, it also puts a limit in it's exploration early on (which sometimes put non-creative assumptions risk that is early on training, hereby making the NN learn quickly in exchange to having "foundational assumptions").
> See the [video here](https://www.youtube.com/watch?v=to-lHJfK4pw&list=PLtBw6njQRU-rwp5__7C0oIVt26ZgjG9NI&index=5)

Insight: Sub-sampling and templates, varying random seeds to the MSA to output multiple proteins and conformations (black-box as salt alternative). If you needed uncertainty quantification or constraints beyond just distances (angle and excluded volumes) or maybe you have a noisy or incomplete distance map(s). Modifying the MSA itself seems core. There are potentials of having an NN here, too. Although in AF2, the geometry formation (the next stage where we construct the 3D coordinates) occurs inside a neural network, not away from it as with the case of AF1. Although it seems like in AF2 the [Cut off]

#### Stage two

Then we take both outputs (Distograms and Angle Matrices), by constructing the full 3D atomic coordinates using either options:

1) Pure Maths (MDS or similar tools) without learning.
2) A blend of physics and mathematics: which is different from a traditional energy function, as it directly "**learns**" from data and principles from physics (Van der Waals forces, and all the other interactions covalent and non-covalent interactions taken into account as constraints), instead of using a less-sophisticated method like MDS. This ensured that the resulting structure was physically realistic, too, also is the approach used by AF1!

*The PHI/PSI angles are used as an initialization to the backbone geometry in this stage, while the distograms are converted to energy potential*

[More to be integrated of value initialization, as well as differing them between LeCun/GLorot and others]
#### Final Output; from the previous stage

![Pasted image 20260129222457.png](Pasted%20image%2020260129222457.png)
[Source](https://biosiva.50webs.org/genomics.htm): A 3D space of a protein,

"**The Atomic Coordinates**": They're the "precise" orthogonal coordinates in 3D positions (X - Y - Z) , all measured in Ångströms (which are really small!) for every atom in a proteins entirety of a tertiary (or quaternary) structure, including the backbone and side-chain atoms. I found it appeasing to introduce them here, as they are the outputs after-all and is the "main" PDB data we're given to think with. We use the PDB ground truth labels indirectly by deriving the distogram from it.

> "Why did we not use the coordinates as substitute to the Distograms?!"

To understand why, we must clarify an important distinction between three similar ideas:
- Atomic coordinates (our data). These are the final result, the solution in [Stage two](#stage-two). They are not used to train the model in any way other than deriving the distogram 
- The primary structure, which is just a string of AminoAcids mentioned in the [1. Primary Structure](#1-primary-structure)
- The [1. True Distogram](#1-true-distogram), This is invariable to the rotations and flips we may make to the protein in a 3D space compared to the infinite and variable precise 3D coordinates, making the former (Distograms) significantly more stable than the latter (precise 3D coordinates). 

So if we inverse, rotate, translate the same structure, the coordinates would change even though it is the exact same structure. Meaning that this representation of proteins has **infinite coordinate representations** precise-coordinate wise, contrast to the beautiful and constant distogram.

*This practically means we had to do the act of deriving or else the network wouldn't have converged at all (Meaning that it cannot see a pattern in the infinite representation of Atomic coordinates). Nothing is "useless".


![Protein-folding-prediction-using-Alphafold-1-18-2048 2.webp](Protein-folding-prediction-using-Alphafold-1-18-2048%202.webp)
[Source](https://www.slideshare.net/slideshow/protein-folding-prediction-using-alphafold-1/251105975): Looking back at the previous image, the entire process should make more sense now (as well as the helpful captions above)

#### Stage three (Optional)

After the converging of the gradient descent on stage 2, using a classical physics simulation to slightly adjust the structure. Removing minor clashes and optimizing local geometry along with other many optimizations. 

The difference between this stage and the second stage is that this is *more fine-grained polishing* (very fine touches) while the former second stage is *more coarse and a global optimization* of the fold. At this third stage, we use the best structure outputted from the previous stage and apply cycles of **energy minimization** and repacking techniques.

#### Disease prediction

Imagine for a second: Two proteins touching each other:

```
MKTAYIAKQ(K)PGLV ... YKV(E)SFIKQ
```

`K:` AA no. 10 which is positively charged
`E:` AA no. 50 which is negatively charged

So if we had AA on position 10 of a protein K, we can safely get another AA with the same charge in it's place, like 'R' , which are both positive (which is also called *a conservative mutation*).

Or else, if that occurs where repulsion meets where there should be attraction (If it were an AA of `A`, which is neutral); The protein misfolds, aggregates and maybe loses its function (No, there is no zombie behaviour for Halloween in such cases!)...

>The stressors of proteins are **heat**, **pH**, and other chemical treatments that make the protein "denure" (to break down), with the exception of one structure:
>
>Having the ***primary structure*** intact during stress provides **renaturation potential**! Implying that for the other two structures (SS and TS), which are greatly affected by the stressors of proteins, can REFOLD back (*also by adding a gentle reintroduction to its normal conditions*) to its correct functional tertiary structure all due to the proteins primary structure. The only thing(s) that can break the proteins primary structures are harsh chemical treatments or specific enzymatic action (proteolysis) to break the covalent bonds.

There are three disease mechanisms (roughly):

1) Loss-of-function
2) Gain-of-Function
3) Dominant-Negative

There are possibilities when proteins do not fold as intended, either due to size mismatches or wrong charges; Maybe due to the energy landscape changing as a result of being in a **different environment** or maybe due to a protein that cannot escape a **kinetic trap**, as in being in wrong conditions and (e.x; no chaperones, the ones helping a protein fold) or maybe **proteasomes** which are quality control (cell level).

*The percentage of diseases coming from Gain-of-Function (GOF), when the protein works too well or does something new and bad) are approximately (24%) of all diseases, while Loss-of-function (LOF), when the protein breaks, are accountable for up to 52% of diseases.*

>(AR) mutations occur at the protein interiors (58%), while only (15%) are in the protein's surface. These are also at the LOF mutations.

Some of which can trigger certain diseases where mutant proteins sometimes gain zombie-like behaviour called Dominant-Negative (DN), and unfortunately, most of these mutations are that of serious diseases; some are: cancers and seizures (GOF). Others are: the Prion disease, some Huntington's diseases // Hemophilia // Cystic fibrosis (LOF) and other such protein-based issues like (DN) whereas it is a contagious mutation causing HMT, Myocilin and some collagen diseases.

So that we got this out of the way, maybe we start thinking using these as an anchor point. Say, a "ΔΔG" to measure a folded protein's stability (i.e: How easily it unfolds and cause trouble) basically a measure of destabilization and such after the protein has folded. Any unstable protein has the potential to unfold, and getts degrades, may also be potential for a LOF mutation.

Now this ΔΔG is also called the change in free energy between two states:

```
ΔΔG = ΔG(mutant) NOT normal - ΔG(wild-type) NORMAL
```

Meaning that ΔΔG (folding) is for stability! But a crucial point to make here is that ΔΔG can be also a binding (interaction). The former (stability) is done after the protein has folded and measures how stable it is, Example:

*Interpretation:*

```
- ΔΔG > +1.0 kcal/mol: Mutation destabilizes → protein misfolds or unfolds more easily
- ΔΔG < -1.0 kcal/mol: Mutation stabilizes → protein is harder to unfold (often GOF signal)
```

The latter ΔΔG (binding) is done also after the protein folding process, but when interacting with something else. So it answers: "Does this mutation weaken or strengthen the protein-protein interaction?". Example:

*Interpretation:

```
- ΔΔG > +1.0 kcal/mol: Mutation weakens interaction → potential GOF or LOF depending on context
- ΔΔG < -1.0 kcal/mol: Mutation strengthens interaction → GOF signal (binds too well to wrong partner)
```

Important misconception: When calculating both ΔΔG's, we're always working with the folded structure (unless your creativity says otherwise). But we're predicting how stable is the final folded state. (Remember? Proteins don't fold once and that's it!)

Now, how much would it take for a mutant to stay in the native state? Or in other words, how much MORE destabilized is the mutant? So the higher the ΔΔG, the more easy it unfolds, the lower it is, the more it resists unfolding (which open up possibilities of GOF).

>ΔΔG (folding) doesn't predict "folding", rather, **it predicts "*unfolding*."**

Architecture wise, there are some already on the shelf computing ΔΔG

1) FoldX and 
2) Rosetta, 
3) and maybe a latest RaSP

*An output may look similar to this (A layered production pipeline, just for demonstration!):*

```
STRUCTURAL ANALYSIS:
- ΔΔG Folding Stability: +2.3 kcal/mol (DESTABILIZING) 
- Position Location: Buried interior (80% burial) 
- Distance to interface: 12.4 Å (isolated) 

RECOMMENDATION: 
- Likely pathogenic (LOF mechanism) 
- Experimental validation: Thermal stability assay 
- Cellular: Check protein expression levels
```

Being aware of the limitations of the current architectures and available tools is crucial, as we need several factors that are deeper than intuition-level:

- aggregation propensity (Forming plaques similar to certain diseases),
- cellular trafficking (Stuck in ER. Used as a DN signal), 
- dynamic properties (Flexibility changing and such),
- binding kinetics (How fast does it unbind)
- cofactor dependence (Metal coordination and its loss)

So they don't explain which forces are broken and such other things. 

*We can take a scenario where it fails to forecast a mutation type:*

```
Scenario: Mutation has ΔΔG = +1.5 kcal/mol (destabilizing)

FoldX says: "This is destabilizing, probably LOF"

But actually:
- IF it breaks a salt bridge in interior then LOF!
- IF it breaks hydrophobic core indicates aggregation (Alzheimer's-like)
  and more...

Same ΔΔG, different diseases!
```

Maybe we need a layering advice: Energy calculations, interface analysis, mechanism-specific logic all on top of the structure that we get from PDB or our model... Thereby increasing complexity, but that's a potential to optimize and hopefully to also find a better approach. *This part feels a bit underdeveloped, though. My condolences*

We delved too much on insights and other introductory parts, but the intuition shall be enough to continue with the implementation of the algorithm itself. This requires basic familiarity with some code, but nothing will pass unquestioned that I myself have even questioned. **We'll zoom in, zoom out, keep seeing both pictures and identify the optimal practices for both Python and Machine learning.**

## Engineering
### Dataset and feature generation ~80%

**UniProt** is a comprehensive protein sequence and annotation DB, containing millions of them. All the other databases point to it! Therefore, two other DBs, like the **UniRef**, we can reference clusters at which the sequences are either 100% or 90% or anything above 50% match with the sequence in question. The purpose in using these other databases is to reduce the probably to find more [Remote or distant homologs](#remote-or-distant-homologs), as we get less-similar sequences.

>**"Uniclust"**, however, goes deeper than a 50% match. That attribute provides us more data to work with, that is if we're willing to take the risk.

*Note on MSA search (skim/read quickly)*

In AF1 case, we are still unsure what and which database they used and which the authors did not. Which is why, we're going to stick with the most likely clustered versions (Uniref90 or 50 at that time). *Although for AF2, it was confirmed that they used Uniclust 30 along with the Uniref90 and implicitly Uniref50 using BFD databases.*

Another note, specifically for [HHblits](https://toolkit.tuebingen.mpg.de/tools/hhblits), and the now-released [Jackhmmr](https://www.ebi.ac.uk/Tools/hmmer/search/jackhmmer): 
There are speculations that HHblits and HMMER tools were used (not explicitly defined) **simultaneously**, but it is confirmed to have been utilized that way in AF2, not AF1.

> Away from this jargon, it's definitely more efficient to not get stuck looking for the perfect tool to use. We pick ones along the way, and that's just good enough.
### Inputs:

*Conceptually*, the first step is **deciding what the NN will predict**: and that is the Pairwise distance distributions between residues (AA's). [The True Distogram](#the-true-distogram)
***That decision forces everything else.*** 

*Practically*, the first code you would write is:
A data pipeline that can turn (sequence + MSA) into supervised labels using PDB and statistics

**PDB's Role**:

- Extract the AA sequence, along with 
- The 3D atomic **coordinates** for each protein chain. 

From coordinates, we derive all of our loss functions, but the sequence itself is not fed directly but is engineered in a way that it is included in the MSA (first row)

*Here's two really cool maps:*

```
PDB coordinates

  ↓
  
derive distograms, torisons, SS, SASA

  ↓
  
NN losses
```

```
sequences
   
   ↓
   
build MSA (now the sequence and its homologs are merged) and convert them to integers

  ↓
  
compute 1D statistics (PSSM, onehot and freq)

  ↓
  
store features

  ↓
  
train NN
```
#### AA sequences

*For a single protein chain:*

*From FASTA*

```
Sequence:  M K V L W Q A L G ...
Length L
```
#### MSA: 

![Pasted image 20260208230432.png](Pasted%20image%2020260208230432.png)
[Source; Veritasium](https://www.youtube.com/watch?v=P_fHJIYENdI); The Multiple Sequence Alignment as previously shown

Using external databases, we search for MSA's using only two tools as opposed to three in AF2;

1) PSI-BLAST to search UniRef90 database. We use PSI-BLAST from an already extracted MSA to build the PSSM later on.
2) HHblits to search BFD. There are tools that negate the Orphan type of homolog when searching for MSA's, but certain others (like this HHblits! and such that we'll be applying) are more sensitive and specifically designed to detect such homologs.

**Note:** *This, as an input, can be a bottleneck in runtime. There should be error handling incase there are no homologs present, other guardrails, etc.*
*Remember, the lower quality MSA we have, the less of a chance we have of modeling effective constraints. [How "Identical" should we aim for?](#how-identical-should-we-aim-for)* 

##### MSA lookup

Let’s say your protein is length **L = 5**

Target sequence: `M K V L W`

*After database search using sequences, you get:*

```python
MSA (N sequences × L positions) 

Seq0 (target): M K V L W 
Seq1           M K I L W 
Seq2           M R V L F 
Seq3           M K V M W 
Seq4           - K V L W
```

Now, we convert these letters as integers, because we need them to count frequencies and build histograms, as well as use them in index lookup tables and compute the statistics to be met in the future. This is partly for the sake of efficiency.

*Encoding amino acids as integers (0–20, 20 = gap):*

```python
M=12 K=8 V=17 L=10 W=19 I=9 R=14 F=5 gap=20
```

*So, an MSA tensor would look like:*

```python
[[12,  8, 17, 10, 19],  
[12,  8,  9, 10, 19],  
[12, 14, 17, 10,  5],  
[12,  8, 17, 12, 19],  
[20,  8, 17, 10, 19]]
```

MSA shape: `(N=5, L=5)`

This is **still not fed directly** to the NN, as the NN cannot consume raw MSA symbols meaningfully. We have one step(s) (haha) to do before that: ***Per-position/residue statistics (1D features of shape `(L, C)`).***

![Pasted image 20260207201810.png](Pasted%20image%2020260207201810.png)
[Source: Veritasium;](https://www.youtube.com/watch?v=P_fHJIYENdI) In the first step, for each column **i** in the MSA (top to bottom), compute frequency of each AA. We do this for all positions:

*Example at position 6:*

```
G (this AA is from our sequence), G, G, D, G, G, G, G (homologs)
```

*Frequencies:*

```
G: 6/7 D: 1/7
```

This gives:

```
20-dimensional vector
```

Outputs a tensor of shape: `(B, L, 20)`

This step answers; *“What substitutions are tolerated here?”* . It sounds like a very solid constraint here using MSA. Good feature engineering

**We then use PSI-blast (PSSM)** to summarize the signal (*probability* of each AA at each single position in a log-odds manner), it other words, it gives an answer to  "How unexpected is AA x at position i?". This is run for each position in the MSA column (this is 1D afterall).

Again: `(B, L, 20)`

Another type of profile we'll be using, which is richer than the PSSM, is called the **HMM** profiles. `(B, L, H)`

For our last 1D feature; **One-hot encoding our sequence by itself,** and it is *just* identity. Has the same shape as the PSSM and MSA frequencies `(B, L, 20)`.

![Pasted image 20260209223824.png](Pasted%20image%2020260209223824.png)
[Source](https://medium.com/@michaeldelsole/what-is-one-hot-encoding-and-how-to-do-it-f0ae272f1179); Onehot encoding illustration vs Label encoding, turning variables into `0` or `1` like a "yes" or "no"

> The `(L, 20)` shape is repeated due to the count of the used "20" Proteinogenic Amino Acids.

We now have **per-residue (1D) features**:

```python
# For each residue i:

X_i = [
one-hot AA (20),   
MSA freq (20),   
PSSM (20),   
HMM profile (~30) 
]

# Concatenated (combined) shape:

(B, L, C₁)
```

*We assume these values are precomputed:*

```python
def build_1d_features(one_hot, pssm, msa_freq, hmm):     
"""    
Returns: (B, L, C1)     

"""    
return torch.cat([one_hot, pssm, msa_freq, hmm], dim=-1)
```

Although we were stressing earlier on the evolutionary information and the idea of "pairs" in MSA, which all cannot be modeled using the one-dimensional data we have now because "folding" is due to non-local interactions, **and the protein structure lives in relationships;** In the one-dimensional data, we describe residue i by itself;

- What AA is here?
- what substitutions (mutations) are conserved here?
- Is this position conserved?

...But 1D **CANNOT** answer:

- Who does this residue interact with?
- Where is it in 3D?
- Is it close to residue j?

Now is the time where we construct the 2D data using the 1D data (while some are not inherently from the 1D). Think of it as expanding features, we're not removing them, but it is implicitly mixed.

>In AF2, there is 1D data used in parallel with the 2D data, but here, it is consumed by the 2D data. **Also in AF2, the MSA is fed raw (and its rows) without statistics.** **So firstly it creates its own 1D features using Raw Single Sequence + Raw MSA (No PSSMs, etc)**, then it earns the 2D features using the "outer product mean" of the MSA (possibly lifting too as done below). Both 1D and 2D also constantly update each other, which sounds way cooler than the static input we currently have in AF1. That is not the motive, though.

### Pairwise per residue pair (2D) features

*So we must reason about:*

```python
(residue i, residue j)
```

We have three features;

1) Lifted 1D features. See the 1D features that were done above? Each of the pairs has now its own 2D profile by lifting the 1D features to be paired with another (for each AA). `(B, L, L, 2*C1)` This is called *feature broadcasting.*

*Illustration of the lifted features:*

```python
pair_context[i,j] =
[
  X_i,
  X_j
]
```

```python
def lift_1d_to_2d(x_1d):
    """
    x_1d: (B, L, C1)

    Returns:
        pair_1d: (B, L, L, 2*C1)
    """
    B, L, C = x_1d.shape

    xi = x_1d.unsqueeze(2).expand(B, L, L, C)
    xj = x_1d.unsqueeze(1).expand(B, L, L, C)

    return torch.cat([xi, xj], dim=-1)
```

2) True pairwise evolutionary features (The co-evolution is here) computed directly from the MSA using classical statistical methods. "If residue `i` mutates from `A` to `V`, residue `j` prefers to mutate from `L` to  `I`". There are **two distinct types**, often stacked together:
- Using Potts or direct coupling analysis (DCA). It infers **direct residue–residue couplings** by removing transitive correlations. "If residue `i` is amino acid `A`, which amino acids does residue `j` prefer to be?" `(L, L, 21×21) often reduced`
- Mutual information (MI)/Covariance. They measure statistical dependence, and linear dependence respectively of columns. “Do these two positions vary together at all?”. These features are **noisy** but cheap and informative `(L, L, 1)`

[an Illustration here is screaming to exist]

>The Potts model was the traditional way of calculating the co-evolutionary signals. It used the traditional methodology of "pseudo-likelihood" maximization to calculate evolutionary signals (inferring direct AA contacts from raw MSAs). In AF2, the NN does just that without it, as mentioned before.

It's also worth noting that we'll be using **Positional encoding**. Nothing fancy as the transformer's position "sinusoidal" embeddings, but it uses very simple positional information; mainly being the sequence index or utilizing sequence separation. We'll go with the latter.

![Pasted image 20260209224105.png](Pasted%20image%2020260209224105.png)
[Source;](https://medium.com/@limemanas0/have-you-ever-wondered-how-a-large-language-model-llm-knows-the-order-of-words-after-it-turns-f598fc3c6cc9) Example illustration using Natural Language Processing

*It helps the NN distinguish:*

```
i=5, j=6   vs   i=5, j=200 (same AA but in a different position)
```

*We use Sequence Separation (topology prior):*

```python
sep = |i − j|
```

That was our last 2D feature! *For each pair (i, j):*

```python
Z_ij =
[
  X_i,                # The two pairs lifted 1D features
  X_j,
  MI / covariance,    # raw correlation
  Potts couplings,    # direct evolutionary signal
  |i - j|             # sequence separation
]
```

Shape: `(L, L, C)`

At this point in the pipeline, **everything is already 2D**. You now have:

```
Z ∈ ℝ^(L × L × C)
```

NEEDED Interpretation:

> For every residue pair (i, j), Zᵢⱼ describes  
> **who *they* are (Lifted 1D)**, **how *they* co-evolve (MSA)**, and **how far apart *they* are in sequence (Position Encoding)**.
## NN architecture ~20%

We first create a Class that inherits the nn.module (so we can do all of its powers like `.train() ` and `.eval()`, and many others mentioned in the Pytorch documentation [here](https://docs.pytorch.org/docs/stable/generated/torch.nn.Module.html)
 
### Fitting to The Problem Statement

We already clarified the reason for approaching the problem using distance maps between AA's and another in [The True Distogram](#the-true-distogram), plus clarified the issues if we were to approach the problem like predicting the entire protein in a 3D representation right away from the output. From the previous data pipeline, it's helpful to reframe the data we got in "images" to understand what we have:

1) Each pixel (i, j) = one residue pair  
2) Each channel = one type of relationship

So the NN problem becomes:

> "Given this image, predict another image  
> where each pixel is a distance distribution."
### Architecture

*Elaboration to something brief before;* 

AlphaFold 1's architecture had a **hard inductive bias** (due to it involving convolutions and a sliding **kernel**, hence being a "CNN-based"); that amino acids were restricted in "seeing" the interaction patterns with their neighbors only in the 2D grid.

>AlphaFold 2 (Transformer-based) removed that restriction, which made it learn that an AminoAcid at the very start of a protein chain might physically touch one at the very end. The **weak bias** of the Transformer allowed it to learn the **true physics** of the protein rather than being stuck in the "local grid" of the CNN. 

Even though CNNs do get enough receptive field once stacked enough, the computational efficiency for such were quite limited, thus, the transformer was used.

At the NN trunk, you start with **channel projection** (hand engineered channels); basically compress the 2D features into a learned space. The reason for the 1×1 kernels is to mix channels and recombine.

*Mixing channels to learn what features matter:*

```python
Z0 = Conv2D(Z, out_channels=H, kernel=1)
```

Then we add residual convolution stack, which are the actual thing driving the learning process. Well, it's only here so the gradient can flow backwards in larger architectures (AF1's, yes). 

*Illustration of the residual blocks:*

```python
Z = Z0
for block in ResNet:
    Z = Z + Conv3x3(ReLU(Conv3x3(Z)))
```

>"What comes after 50 or so blocks of this trunk?"

```
H ∈ ℝ^(L × L × D)
```

They encode things like: "Likely same helix, anti-parallel strand, packing constraint, rigid vs flexible region"
This is **representation learning** (Embeddings are fierce)

![Pasted image 20260205165616.png](Pasted%20image%2020260205165616.png)
[Source](https://docs.3lc.ai/3lc/latest/user-guide/column-types/embeddings.html): An embedding space from the CIFAR-10 dataset. They were reduced to `3D` using PaCMAP, so they're supposed to be even bigger!

> "Can you reuse an embedding layer?"

**Yes, technically.** But if you've done it earlier, it would have not carried the signal you'd think it will. The “meaning” of an amino acid comes from **how it mutates across homologs** (which is why AF1 puts its modeling power after MSA processing).

Mind you, this is no longer interpret-able. This happens to be the line of black-box that was briefly discussed earlier in the [The Problem Statement](#the-problem-statement) section. 

*Also to have "padding" on for Convnets, because having same shapes is a necessary design choice, but not strictly, for the residual connection to work. ML-strict!*
#### Loss Functions and their shapes:
| #   | Component                           | Loss Function             | Output                           | Shape                   | Notes                                                                                       |
| --- | ----------------------------------- | ------------------------- | -------------------------------- | ----------------------- | ------------------------------------------------------------------------------------------- |
| 1   | Distogram                           | Cross-entropy             | P(d_ij ∈ bins)                   | [L, L, N_bins]          | 64-bin distance distribution; exact weights not published, although the distograms dominate |
| 2   | Angles (Phi, Psi)                   | Von Mises / Cross-entropy | P(phi_i, psi_i)                  | [L, bins_phi, bins_psi] | Torsion angle prediction                                                                    |
| 3a  | Auxiliary: Secondary Structure (SS) | Cross-entropy             | P(helix, strand, coil)           | [L, 3]                  | Per-residue classification                                                                  |
| 3b  | Auxiliary: SASA                     | Mean squared error        | Normalized solvent accessibility | [L, 1]                  | Regression target                                                                           |

#### Outputs

In the distogram outputs, we do distance bins classification (grouping predictions) and settling for "Good Enough" instead of regression (because distances are noisy and multiple modes exist);

```
0.1A, 0.2A, 0.3A bins and so on till a contact threshold of 8A.
```

Then after they get far enough, the bins become more coarse `8A, 9A+` till we reach the `20A` threshold for a "functionality irrelevant" state between two AA pairs.

*A reminder,* we're not only are we predicting distances that are "close enough" (bins), we also **do not** produce a 3D output predicting the exact protein shape in the NN. This is still training and behind outputs.

*For the distogram:*

```python
logits = Conv2D(H, out_channels=num_bins, kernel=1)
P = softmax(logits, dim=bin)
```

*The losses:*

```python
L = (
    CE(dist_logits, true_dist_bins)
  + λ1 * CE(torsion_pred, true_torsion)
  + λ2 * CE(ss_pred, true_ss)
  + λ3 * MSE(sasa_pred, true_sasa)
```

### Stage two: Using guided minimization

Here, an algorithm is used (specifically a version of gradient descent: L-BFGS algorithm), that used this potential surface as guidance to fold the protein and find the lowest "energy", outputting a 3D structure. In other words, this stage optimizes the backbone Cα positions (X - Y - Z) to satisfy the distance predictions when training. Three processes occur here:

1) Multiple starting points, implying that the algorithm was run multiple times with different 3D configurations of the protein (all random). This explores a wide range of possible folds (related to the previous insight?!) then it performs an
2) Iterative minimization, adjusting the AA positions to get the most steeply decreased potential function (basically epochs for you ML engineers), and finally
3) We choose the best candidates after converging. They are the lowest-energy structure from multiple starting runs implied on the first step of this algorithm. 

### Stage three: Optional 

refinement using Rosetta REF15 energy function.
## Inference and recap (To be refined)

Given a novel, non-seen-before protein sequence: `MSVTQRFIAKQ`(1)
We generate a MSA (2) using an external DB (PSI-BLAST) and compute 1D statistics to lift to 2D using broadcasting. Then, after the output of the NN, we use the distance map (distograms) and torsion angle initialization to construct a 3D, folded protein.

Note that AF1 is far away from being an E2E architecture, because of the external MSA generation and requiring GradientDecsent (GD) optimization rather than directly predicting coordinates.

[Continue]

## To be added:

- Doodles on the images themselves (studying purposes obviously)
- Instances of outer scope interdisciplinary knowledge are to be integrated when needed (should serve as fun "tips" for your physical and mental life as well)
- Better Author Attribution
- All the "continue"'s
- Linking factual information and separating it from speculative theory (hard to do if we're talking about AF1)
- Including the original paper in attempt of paper replication, not truly word-for-word but close enough
- Align the topics to their respective headings. Some are scattered!
- Many other images for demonstration and illustration. Maybe a self-made GIF to clear up both the bigger and smaller pictures. Heavy intermediate factors may be distracting!
- Foldit competition and more relevant documentation are WIP along with RL enhancement

Important links:

Citation: 

**John Jumper, et al. (2018). _AlphaFold: A Protein Structure Prediction System_. The abstract book of the 13th Critical Assessment of Techniques for Protein Structure Prediction (CASP13).**

Varadi, M. et al. "AlphaFold Protein Structure Database in 2024: Providing structure coverage for over 214 million protein sequences." _Nucleic Acids Research_ gkad1011 (2023). DOI: 10.1093/nar/gkad1011.

Zhang G, Liu C, Lu J, Zhang S, Zhu L. The Role of AI-Driven De Novo Protein Design in the Exploration of the Protein Functional Universe. Biology (Basel). 2025 Sep 15;14(9):1268. doi: 10.3390/biology14091268. PMID: 41007412; PMCID: PMC12467925.

Senior, A.W., Evans, R., Jumper, J. et al. "Improved protein structure prediction using potentials from deep learning." _Nature_ 577, 706–710 (2020).

**Jumper, J. et al. "Highly accurate protein structure prediction with AlphaFold." _Nature_ 596, 583–589 (2021). DOI: 10.1038/s41586-021-03819-2**

Abramson, J., Adler, J., Dunger, J. et al. Accurate structure prediction of biomolecular interactions with AlphaFold 3. _Nature_ **630**, 493–500 (2024).

Interesting for later:

http://fhalab.caltech.edu/?page_id=171#page-content

...et al, too:

https://discovery.ucl.ac.uk/10142031/1/NatMethOpinionRevised-Final.pdf

https://pmc.ncbi.nlm.nih.gov/articles/PMC11348012/

https://medium.com/@satishlokhande5674/what-information-does-alphafold-use-from-the-protein-data-bank-44048e672100

https://github.com/google-deepmind/deepmind-research/tree/master

https://pmc.ncbi.nlm.nih.gov/articles/PMC11319189/

https://pmc.ncbi.nlm.nih.gov/articles/PMC9710616/

https://www.youtube.com/watch?v=NN_uRCH7mrQ

https://pmc.ncbi.nlm.nih.gov/articles/PMC5588872/

https://lmu.pressbooks.pub/conceptsinbiology/chapter/protein-structure/

https://pmc.ncbi.nlm.nih.gov/articles/PMC5588872/