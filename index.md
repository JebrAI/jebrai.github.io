
# Scope

This document should serve anyone with curiosity towards the 
- Basic protein knowledge (No promises)
- core concepts of the architecture,  
- Motivation behind each idea
- Problem statements along with their solutions, by fitting the (refined) data to a (custom) model. A fancy way of saying
- and lastly what could be used to extend the idea itself of AF1 (requires some expertise on both fields). As well as questions that have incurred to me personally while attempting to understand the architecture, motivation and how are all things related. These are thrown throughout the document as "insights". It is favorable for the reader to absorb it however they like.

Then, everything is wrapped with a "before in each component pass forward" and after to enhance the engineering aspect, as well as solidify our understanding towards each components role.

### Extension

AF1 is the stepping stone towards a fundamental problem we have on a different field of interest, but we should only recognize the patterns used here as one maybe able to discover an efficient AF1 that's better than it's "upgrades".
While I thoroughly study AP2 and other architectures, the notes will be left here as possibilities for extension for oneself to exercise "How can we improve this" so the reader can delve and realize they're passing an important, guaranteed (something that is built upon, AP2 and AP3 for example. There are possibilities also) questions that are answered in later architectures.

Notice: This may be published before polished, maybe even before folding completely. 
Updates are gradual and links shall be provided along many other hiccups that should be adressed.
Any feedback of any form is welcome.

## Brevity

To avoid repeating "AlphaFold 1 by DeepMind"; AF1 is the norm. 

AminoAcid abbreviation of "AA" which is used synonymously with "residue" or "res", 

along with the abbreviation of Angstroms: "Ang."
....

Reinforcement Learning: "RL"

ProtienDataBank: "PDB"

Others are made on-spot.

## Foreword

What AF1 basically did:

'Alphafold uses advanced biotechnology and AI to help determine longer protein structures over a shorter period (implying the traditional, still used x-ray of proteins called crystallography...)'

One example that have just come to my mind: 

The use of crystallography and OTHER techniques in identifying protein structure. They're a accurate way of figuring out the structure of proteins, but have my I ever thought of OTHER techniques, like Nuclear Magnetic Resonance, spectroscopy, Cryo-EM?) Well, if you go even deeper in that branch, you'd also realize the potential of ML application towards traditional techniques like crystallography and these OTHER techniques. 

See the hint? 
There can be a ML algorithm as an "Aid'er//Helper algorithm" in these techniques themselves, maybe even AF1 having an "Aid'er" Meaning that besides a ML framework taking the core idea and digest (AF1), we'd have it as a secondary component that aids in the function of the core algorithm.

It also takes some form of delusion, too (mind you). As this is useful as a possibility opener if one is truly passionate about proteins and can open doors one couldn't have imagined (yes, yes bla bla) as long as it doesn't distract one from pursuing one single technique to their subjective creative limits.

---
### Small biology lesson:
### **The Problem Statement:**

Proteins are made from instructions made by DNA and RNA (the messenger of DNA). It, the protein, then folds to a 3D structure to do a specific function in the body (i.e. native conformation).

<img width="500" height="375" alt="DHRS7B_homology_model" src="https://github.com/user-attachments/assets/7b493534-0e37-43d4-a77e-0e017bfc5186" />



Source: https://en.wikipedia.org/wiki/Homology_modeling an image of the DHRS7B protein (dehydrogenase/reductase 7B) created using Ribbon diagrams and rendered with PyMOL

AminoAcids (AA's) are made up of a carbon atom with a carboxyl and amine groups each to the center (the carbon). The last bond would be either of the side chains forming the 20 AA's that exist in nature, not taking into account the unknowns, and the rare AA's. They're encoded in Alphabetical names, such as "`U`" for the Selenocysteine AminoAcid (there are some not encoded with starting names). The unknown one(s), or harder to distinguish between other AminoAcids are as following:

1) `B` (Difficult; may be a `D` or `N`),
2) `Z` (Difficult; may be `E` or `Q`) 
3) `J` (Difficult; may be `L` or `I`) and finally:
4) `X`(Undetermined or any unknown AA).

Along with the rare AA's, which are the AA's no. 21 and 22 respectively:

1) `U` and 
2) `O`

Some of these are not needed in human trials, for example the `O`, found in certain archaea and bacteria unless there are specific needs or research to be conducted on these organisms
You can call the 22 AminoAcids "Proteinogenic", which means they are used for building the protein from sequencing and folding. It's important to note that there are over 500 naturally occurring AminoAcids that have been identified, in which case are called "Non-proteinogenic" AminoAcids. They are beyond the scope of proteins and the alphabetical order, as in they do not serve the function as to become proteins, but they serve other biological functions There are thousands others used to be created artificially by chemists for drug discovery and such, but RF diffusion, are called the "Building Blocks"

![507294015-d8e68cba-d0d3-46e2-a5bb-3943f5ea3df9](https://github.com/user-attachments/assets/7dd80a28-4b25-4e16-89c1-0c886c54a8c2)

Source: https://www.ml6.eu/en/blog/esm3-the-frontier-of-protein-design

Sequence determines structure, structure determines function --- and .. Oh, wait:

So we hit a "RF diffusion" spot here, where can are able to "create" proteins for specific functions (extremely cool I know!) that we dictate in a model for it to produce a 3D image of the protein design. They use the Proteinogenic 20 (sometimes the other two) for this, not creating new AA's!
Check out bakerslab: https://www.bakerlab.org/2023/07/11/diffusion-model-for-protein-design/

Continuing;
We have DATA from the infamous ProtienDataBank (PDB), in which gotten from the X-ray crystallography has an environment which has:
 - Non-physiological PH or salt conc. (These were used to force crystallization)
 - Low temperatures (sometimes)
 - Absence of binding partners or cofactors that normally are present
 - Dehydration, etc

### PDB bias:

Protein folding is both a deterministic and stochastic process. Its determinism comes from the AA's specific sequence, as ***most*** proteins with the exact same AA sequence have obviously the same shapes (scientifically the "lowest-energy state"), while the environmental factors and random thermal motions (pH, ions, ligands, etc) add the stochastic elements in the folding process. 

AF1 uses the deterministic side of the equation. In other words, they assume stable environmental and specific laboratory conditions. At the worst case scenario, there is misfolding, and all the bad things could happen if a protein is introduced to an extreme environment,

At every other scenario, even a fever of 39c or so, there is little to no effect to the protein and stochasticity which implies a more deterministic outlook that we have. Only after that point, though, does it compromise proteins and its folding process, and the chances increase from thereon.

The uncertainty in AF1 predictions, where even 1-2Ang. is considered uncertain, is much larger than a 2C shift in temperature. Though marginal activity may drop but still has no significant effect, maybe even more "flexible" and within biological tolerance (The effects that I'm currently aware of. Don't solidify that as a foundational barrier to your brain).

So, unless we're making protein fold in "possible" scenarios, this may not matter as much. It's great to keep this in mind, though, as maybe you'd want to make a fever model or a very unethical application in which I do not endorse, maybe a certain environment-based protein folding process in the future that adds on to that layer (or create a "new AF1", really.). AF1 was optimized for the deterministic side, and it's the most efficient and used case for drug discovery and other issues with the "lowest complexity" that we're currently aware of
[Incomplete]


Insight: Reinforcement learning


![image](https://github.com/user-attachments/assets/e7d18579-a635-48be-ae45-f687d1aacba1)


Source: Reinforcement Learning by Maxim Lapan (Cool book)

How can you apply reinforcement learning here? Because it has both stochasticity (at the beginning with possible environmental changes), and also has deterministic phases in later stages by decreasing the epsilon-greedy parameter. But that's only intuition, and not truly touching on the main idea of the document! Like increasing stochasticity the more temperature (a Boltzmann parameter now?!) we rise up in the folding process and subsequently decrease the determinism factor.

You can start small, because the reward signal maybe is the challenging part. As some are sparse and some are goal-based rewards (which require knowing the answer), 

But if we think about it as a non-originator algorithm (not being the core algorithm), RL may shine in refining coarse predictions from AF1 itself, that part that AF1 missed which is the stochastic parts, in which we have:

- **State:** Current structure from AlphaFold 
- **Action:** Local perturbation (adjust a region) 
- **Reward:** Energy decrease + structural validity 
- **Policy:** Learn to make smart local adjustments that respect physics!

As we mentioned though, this is for another talk!


### What we have so far:

Two key data:

- Coordinates of 3D positions (X - Y - Z) all measured in Ångströms (Ang.) which are really small! For every atom in the protein. Along with occupancy (likelihood of protein binding to a specific site), temperature factors and element names. Coords are also called "Cα coordinates".
- AA sequence: E.x: `MKTAYIAKQRPGLV` which isA mapping to UniProt for proteins and GenBank for RNA's

The always available data:

- Secondary Structure (SS) assignments (derived from 3D structures for a deterministic assignment, and derived from the AA sequence for a probabilistic assignment)

<img width="360" height="140" alt="507294351-e2c2c0d5-68a1-4bc0-b7d5-94c49fc2a768" src="https://github.com/user-attachments/assets/10080a2b-5b6b-469d-bb02-d983757fddf3" />

Source: https://faculty.uml.edu/vbarsegov/research/atob.html\

- Experimentation methodology: Xray, NMR, cryo-em
- Resolution quality metric

The sometimes available data:

- Ligands/cofactors
- biological unit info
- Experimental conditions PH and temp (probably not in the obviously freezing X-ray)
- Homologous sequences (In a separate DB, not PDB)
- MSA: (Separate DB, but simply how hemoglobin for example looks for every specifies there is to be found by the DB organizers)

![507294500-46d3a87f-ee1a-40c3-967b-b2993fee75d8](https://github.com/user-attachments/assets/46db833f-66b2-4b8e-af49-8a86f19a6557)


By Miguel Andrade at English Wikipedia - Transferred from en.wikipedia to Commons., CC BY-SA 3.0, https://commons.wikimedia.org/w/index.php?curid=3930704

The rest still have potential if we had a thinking session (computed ourselves and if we need it):

- Contact maps (derived from coordinates) which are basically binary maps of touching or not 0 1000 1 0101!
- Distance matrices: (Also coordinates)
- Evolutionary profiles (derived from something other than coordinates: MSA!)

## Implementation and Intuition:
### Bigger picture

There is an input, and there is an output (Stage 1). Then, there is a Structural Construction stage, where we utlize physics-based constraints on all the inputs from stage 1 to get the final 3D coordinates. Then, optionally, we attempt to reduce the energy to its lowest possible energy state (of course, based on what the model can handle with its current complexity!) using also a physics-based energy minimization. This third phase uses the 2nd stage as input and has an output of a refined 3D coordinates implying a yay!-level-lower-energy state of the protein predicted.

There are Physics-Based refinements in AF1:
But AF1 used a physics module, where it uses the energy potential with the NN (Neural Network) outputs. It tried to minimize the energy using physics!

But since ***most*** architectures aren't using the "physics" aspect as modules anymore, as that they avoid explicitly encoding the physics by gaining enough PDB examples, one can safely say that there is a way to skip that part to learn the lowest-energy state (the folding process) and go pure pattern recognition from MSA's and Structural data. This is a possibility to note.

AF1 didn't represent the minimum complexity of modeling proteins, it achieved a rather significant advancement than template-based modeling (same protein == same structure) and Ab initio which uses physics energy functions to simulate folding (in which also rarely worked well, but hey, potentials are everywhere). Basically, you can say AF1 is the proof-of-concept that one can apply DL techniques on proteins, with other architectures using the same dataset of PDB, also using different methods such as eliminating MSA and such.

Maybe all it needed was more essentials and lesser complexity.

### Training

#### To compute the targets:

- AA sequences, 
- Cα coordinates (X - Y - Z space of each AA in a 3D space)

And we precompute the following, these are fed to the NN to enhance learning tremendously.

- MSA: In simple terms, we see if two AA stay conserved across different specifies and other AA that may have co-evolved. Meaning that they mutate together, and that often signals that they're often close in physical contact when folded to 3D structure. This is crucial input to the NN.

> Any confusion? There is a section for that below.

Therefore, we can conclude that we have two input features: The AA sequences along with the MSA. The Cα coordinates are only given throughout training as ground-truth data that we derive "The targets" from, not during inference.

#### Targets (Loss functions)

So the model has to learn to map the sequences (along with MSA) to structures (somehow!)
We use the PDB ground truth to minimize the loss function and update the NN weights. Here, we use two matrices computed from the Cα coordinates to use as target values (ground truth). They're pure mathematics:

1) Distance matrix (Remember Pythagoras? Now it's a 3D version of that!!). The distances between two residues are stored in a matrix till we compare the distance between each AA and other in the SAME PROTEIN. Hence, we call it "a distance between specific atoms" // "distograms" or distance distributions:

Example:

```
   M    S    V    T    Q 
M  0  3.8  7.2  12.1  15.3 
S  3.8  0  4.1  8.9   14.2 
V  7.2 4.1  0   5.3  10.1 
T  12.1  8.9 5.3  0  4.8 
Q  15.3 14.2 10.1 4.8  0
```

So they're distances between pairs of AA's. You can see that the distance between AA "`M`" and itself is zero, while other pairs facing the same pattern: Hinted by a diagonal line. This was an empirical way to quantify the protein structure

The NN can conclude, eventually:
Close residues (3-4 Å) then a match in helix shapes
Variable distances then susceptible loops
Far apart in sequence but close in space then a possible folding interaction

You can optionally switch them to binary Contact Maps when hit a range:

1 = if distance is below 8Ang.
0 = Above 8Ang.

Example:   


```
   M  S V T Q   
M  0 1 0 0 0   
S  1 0 1 0 0  
V  0 1 0 1 0   
T  0 0 1 0 1   
Q  0 0 0 1 0
```

So if we think about outputting the 3D structure itself, we hit a roadblock that our NN cannot distinguish from easily.

2) Angle Matrix: different than the distance matrix, as in that it it uses the backbone and side-chain dihedral angles. Basically the rotational degrees of freedom that define the SS (Directly) of Alpha and beta sheets! Phi and Psi are also called Backbone Torsion Angles.

![Dihedral-angles-in-glutamate-Dihedral-angles-are-the-main-degrees-of-freedom-for-the 1](https://github.com/user-attachments/assets/17c9bd4d-8fd5-4ef7-be71-c03dae1cc831)

Source: https://www.researchgate.net/figure/Dihedral-angles-in-glutamate-Dihedral-angles-are-the-main-degrees-of-freedom-for-the_fig4_44651362

Angle matrices and secondary structure related, while the tertiary structure, meaning the distance map itself, is more of the overall 3D fold. How all residues position relative to each others, and all interactions across proteins.

There exists another depth of structure, to keep in mind;

Quaternary structure: Multi-protein complexes. Meaning multiple proteins chained together, making a complex (out of scope of AF1). Haven't delved into it, but may be of use later.

<img width="600" height="764" alt="71225d815cafcc09102504abdf4e10927283be98" src="https://github.com/user-attachments/assets/7c04a661-d271-4157-a5fe-877b2145c4d4" />


Source: https://www.khanacademy.org/science/biology/macromolecules/proteins-and-amino-acids/a/orders-of-protein-structure (Honestly an amazing demonstration for the bigger picture view, maybe have some terminologies in brackets like pleated sheets should be "Beta Pleated sheets" but that's only to aid beginners in learning structure terms).

There was another component that needed to be addressed:

3) The (Sec)ondary(Struct)ure (SS) profile, often derived from the AA sequence. SS is generated by running PSIPRED or similar on the sequence (likely computed from coordinates, in AF1, though), then outputs a probability of each position being a H(helix: spiral)/E(strand)/C(coil: strand but unstructured form). This aids the NN to learn, **but we don't specifically output them, nor do we specifically input them to the NN itself!** We use these as auxiliary losses or an additional training signal during training with a very marginal weight provided (not affecting the outputs as much as other loss functions like Distance Matrices and the Angle Matrices).
#### Output:

#### Stage one

...are the two targets mentioned: an Angle matrix and a Distance map. Then optionally, the SS are derived from those distance maps themselves using other algorithms like DSSP to assign H/E/C by analyzing angles of the distance maps (which is AFTER the prediction of the NN). If we wanted to know and predict the SS before the model outputs a 3D structure, we use the AA sequence and ingest it to a "PSIPRED" or a "S4PRED" to get a prediction. Whichever fits the speed/accuracy respectively.

Insight: Subsampling and templates, varying random seeds to the MSA to output multiple proteins and conformations (black-box as salt alternative). If you needed uncertainty quantification\ constraints beyond just distances (angle and excluded volumes) or maybe we have noisy or incomplete distance maps. Modifying the MSA itself seems core. There are potentials of having a NN here, too.

#### Stage two

Then we convert both outputs (Distance and Angle Matrices), by constructing the full 3D atomic coordinates using either options: Neural-Network predicted "Potential surface" (fancy name for the two outputs we have),

1) Pure Math's (MDS or similar tools) without learning.
2) A blend of physics and mathematics (The NN output): which is different from a traditional energy function, as it directly learns from data and principles from physics (Van der Waals forces), instead of using a less-sophisticated method like MDS. This ensured that the resulting structure was physically realistic, too. This is the approach used by AF1!

Using guided minimization, which is an algorithm (specifically a version of gradient descent: L-BFGS algorithm), used this potential surface as guidance to fold the protein and find the lowest "energy", outputting a 3D structure. Three stages occur here:

1) Multiple starting points, implying that the algorithm was run multiple times with different 3D configurations of the protein (all random). This explores a wide range of possible folds (related to the previous insight?!) then it utilizes:
2) Iterative minimization, adjusting the AA positions to get the most steeply decreased potential function (basically epochs for you ML engineers), and finally
3) We choose the best candidates after converging. They are the lowest-energy structure from multiple starting runs implied on the first step of this algorithm. 

Disappointedly, there are no pure quantum-level physics involved, as the energy function is semi-empirical (derived from known structures)
### Stage three (Optional)

After the converging of the gradient descent on stage 2, using a classical physics simulation to slightly adjust the structure. Removing minor clashes and optimizing local geometry. 


---

#### For confusion on MSA:

Take a single protein for example, we'll call it hemoglobin as a scientific name over "blood":

`MKTAYIAKQRPGLV`

So if we say that "M" and "T" are pretty close in that 1D sequence, it may be temping to say they are close to each other in 3D. but proteins can surprise you with how far they actually are.

But, for this specific case, these are called structural significance. As in, M and T maybe are not close in 3D space, but they are important for the structure itself. Don't mix the two ideas.

Imagine the same protein in 1,000 different species: ``` 

Human: MKTAYIAKQRPGLV... 
Chimp: MKTAYIAKQRPGLV... (99% identical) 
Mouse: MKTAYISKQRPGLV... (changed position 7: A→S) 
Fish: MKTVYISKERPGLV... (changed positions 4,7,9) 
Bacteria: MKTVFISQERPKLV... (many changes) ```

If positions 7 and 150 are in contact in 3D (We x'rayed it):

When position 7 mutates A to S (called polar change), position 150 (in which it is too far to be writen sequentially here) also mutates to maintain interaction (here it is again, co-evolution!)

### Orphan proteins quick introduction

These are proteins without homologs (No MSA depth). Here, the sequence itself becomes the only input available for the NN, and also means that AF1 gets zero coevolutionary information. There are other algorithms utilizing language models like RGN2 and ESMFOLD and trRosettaX-Single to predict structures. One may think of combining the two, or utilizing the other when there are no homologs available. Other may think of generating Synthetic MSAs (beneficial for Alphafold and its variants) like GhostFold and such. So anything out of MSAs is definitely out of AF1 (unless you're seeing something that I cannot see).


### Attraction and repulsion, size, and mutations:

There are things called attraction forces coming from salt bridges, we need positive and negative charges together so the entire protein can be stable. So as long as we have that charge balance, everything can become stable.

Imagine for a second: Two proteins touching each other:
`MKTAYIAKQ(K)PGLV ... YKV(E)SFIKQ`

`K:` AA no. 10 which is positively charged
`E:` AA no. 50 which is negatively charged

So if we had AA on position 10 of a protein K, we can safely get another AA with the same charge in it's place, like 'R' , which are both positive (which is also called a conservative mutation).

If it were an AA of 'A' (which is neutral), it may have consequences of misfolding the protein!!

Or else, if that occurs where repulsion meets where there should be attraction; The protein misfolds, aggregates and maybe loses function (No, there is no zombie behaviour for Halloween for such cases!)...

Another catch: Disease prediction.


There are three disease mechanisms (Roughly):

1) Loss-of-function
2) Gain-of-Function
3) Dominant-Negative

The percentages are subjectively approximated, but conveys the meaning nonetheless

There are possibilities when proteins do not fold as intended, either due to size mismatches or wrong charges; Maybe due to the energy landscape changing as a result of a different environment or maybe due to a protein that cannot escape a kinetic trap, as in being in wrong conditions and no chaperones or maybe proteasomes which are quality control (cell level).

The percentage of diseases coming from Gain-of-Function (GOF), when the protein works too well or does something new and bad) are approximately (24%) of all diseases, while Loss-of-function (LOF), when the protein breaks, are accountable for up to 52% of diseases.

Keep in mind that (AR) mutations occur at the protein interiors (58%), while only (15%) are in the protein's surface. These are also at the LOF mutations.

Some of which can trigger certain diseases where mutant proteins sometimes gain zombie-like behaviour called Dominant-Negative (DN) (That's the Halloween spirit!) and unfortunately, most of these mutations are that of serious diseases; some are: cancers and seizures (GOF). Others are: the Prion disease, some Huntington's diseases // Hemophilia // Cystic fibrosis (LOF) and other such protein-based issues like (DN) whereas it is a contagious mutation causing HMT, Myocilin and some collagen diseases.

So that we got this out of the way, maybe we start thinking using these as an anchor point. Say, a "ΔΔG" to measure a folded protein's stability (i.e: How easily it unfolds and cause trouble) basically a measure of destabilization and such after the protein has folded. Any unstable protein has the potential to unfold, and getts degrades, may also be potential for a LOF mutation.

Now this ΔΔG is also called the change in free energy between two states:

`ΔΔG = ΔG(mutant) NOT normal - ΔG(wild-type) NORMAL`

Meaning that ΔΔG (folding) is for stability! But a crucial point to make here is that ΔΔG can be also a binding (interaction). The former (stability) is done after the protein has folded and measures how stable it is, Example:

**Interpretation:**

- ΔΔG > +1.0 kcal/mol: Mutation destabilizes → protein misfolds or unfolds more easily
- ΔΔG < -1.0 kcal/mol: Mutation stabilizes → protein is harder to unfold (often GOF signal)

The latter ΔΔG (binding) is done also after the protein folding process, but when interacting with something else. So it answers: "Does this mutation weaken or strengthen the protein-protein interaction?". Example:

**Interpretation:**

- ΔΔG > +1.0 kcal/mol: Mutation weakens interaction → potential GOF or LOF depending on context
- ΔΔG < -1.0 kcal/mol: Mutation strengthens interaction → GOF signal (binds too well to wrong partner)

Important common misconception: When calculating both ΔΔG's, we're always working with the folded structure (unless your creativity says otherwise). But we're predicting how stable is the final folded state. You see, the folding process isn't a one-time event. Proteins are constantly flexing, temporarily unfolding at the edges, refolding, WOW!

Now, how much would it take for a mutant to stay in the native state? Or in other words, how much MORE destabilized is the mutant? So the higher the ΔΔG, the more easy it unfolds, the lower it is, the more it resists unfolding (which open up possibilities of GOF).

So, to sum it up, ΔΔG (folding) doesn't predict "folding." It predicts "unfolding."

Architecture wise, there are some already on the shelf computing ΔΔG!

1) FoldX and 
2) Rosetta, 
3) and maybe a latest RaSP

An output may look similar to this (A layered production pipeline, just for demonstration!):

STRUCTURAL ANALYSIS:
- ΔΔG Folding Stability: +2.3 kcal/mol (DESTABILIZING) 
- Position Location: Buried interior (80% burial) 
- Distance to interface: 12.4 Å (isolated) 

Rationale:
- Strong destabilization signal 
- Interior position typical of LOF 
- Not at interface (rules out DN) 
- No stabilization (rules out GOF) 

RECOMMENDATION: 
- Likely pathogenic (LOF mechanism) 
- Experimental validation: Thermal stability assay 
- Cellular: Check protein expression levels

Anyways, being aware of the limitations of our current architectures and available tools is crucial, as we need 
- aggregation propensity (Forming plaques similar to certain diseases),
- cellular trafficking (Stuck in ER. Used as a DN signal), 
- dynamic properties (Flexibility changing and such),
- binding kinetics (How fast does it unbind)
- cofactor dependence (Metal coordination and its loss)

So they don't explain which forces are broken and such other things

Scenario: Mutation has ΔΔG = +1.5 kcal/mol (destabilizing)

FoldX says: "This is destabilizing, probably LOF"

But actually:
- IF it breaks a salt bridge in interior then LOF!
- IF it breaks interface salt bridge then a DN (worse!) 
- IF it breaks hydrophobic core indicates aggregation (Alzheimer's-like)
- IF it creates steric clash may be a kinetic trap
- IF it disrupts active site then there is loss of catalysis

Same ΔΔG, different diseases!

Maybe we need a layering advice: Energy calculations, interface analysis, mechanism-specific logic all on top of the structure that we get from PDB or our model... all increasing complexity but that's a potential right there to optimize and hopefully find a better approach. This part feels a bit underdeveloped, though. My condolences

## Dataset and preparation (~80%)

### Inputs:

1) AA sequences:

2) MSA: We use HHblits to search the UniRef90/Uniclust30 databases, along with PSI-blast also, to generate the MSA. Potts models are then created from the MSA, then fitted on the generated MSAs using pseudolikelihood to finally generate the coevolutionary features. JackHMMR is used in AF2, though.

This, as an input, can be a bottleneck in runtime and we should have error handling incase there are no homologs present. The lower quality MSA we have the lesser accurate the model may be. Although some algorithms have completely removed this component with slightly diminished accuracy

### Targets:

1) Distance maps 
2) Angles
## NN architecture (~20%)

We first create a Class that inherits the NN.module (so we can do all of its powers like `.train() ` and `.eval()`, and many others mentioned in the documentation [[here]]. 
 
## Problem statement and fitting 


We already clarified the reason for approaching the problem using distance maps between AA's and another, plus clarified the issues if we were to approach the problem like predicting the entire protein [P1] in a 3D representation right away from the output [P2]. 

In protein distance prediction, we do distance bins classification (grouping predictions and settling for "Good Enough" instead of regression. So, we predict the not the exact distances, but an approximation of them: 

0.1A 0.2A 0.3A bins and so on till 8A; because contacts matter in proteins 
Then after they get far enough, the bins become more coarse 8A, 9A etc till we reach the 20 angstorm threshold for a "functionality irrelevant" state between AA's.

So not only are we predicting distances that are "close enough "
 (Bins), we also do not produce a 3D output predicting the exact protein shape, but were outputting a distance map between each two AA's. 

## Embeddings

We then create an embedding layer, which basically serves as our "means" [*demonstration] ,

1) So we give the embedding space an index (AA) 
2) We give it the index we DO NOT want to update (like padding idx which is index(20) in our VOCAB). If we remove this part, we'd be training the padding tokens and the model would think there are patterns in such cases: 

But here's the catch: Maybe one lesser protein still has potential in making useful pattern, in such case we'd basically be adding a "feature", not a "bug" in our training. For example: adding soft masking with learned embedding; we provide the padding with a learnable identity (i.e not shouting "ignore me!!", but represents a "no amino acid" index).

Back at the embedding space, we had created ONE embedding for EACH AA. One way to ensure the model has enough "learning capacity" is by introducing more embedding dimensions. So we create a 64 dimensional embedding space for EACH amino acid. 

So if we had 2 proteins, max length (between them) is 8 AA's:

```python
[[0, 1, 2, 3, 4, 20, 20, 20], #Take note of the first zero on protein 1
  [0, 1, 2, 3, 4,  5,  6,  7]] 

# batch_size = 2
# seq_len = 8
# Their shape is (2 , 8)
```

Then the forward pass of the embedding layer:

```python
embedding.weight[0, :] = [0.23, -0.45, ..., 0.67]  # 64 numbers for amino acid A, which is the zero index protein one. 
# Shape has become (2, 8, 64)
```
### Masks

Now, we apply the masks to identify which lookup tables (embeddings) should be trained. In this notebook, we go with hard masking as mentioned before. But to those curious to apply soft masking (learnable identity), it'd go somewhere along the following lines: 

```
self.pad_embedding = nn.Parameter(torch.zeros(embed_dim))
# embeddings = torch.where(
#     masks.unsqueeze(-1).bool(),
#     embeddings,
#     self.pad_embedding
```

Hard masking, however, is applied by multiplying (zeroing) the padding tokens by zero and all the other values multiplied by zero, which are actual AA's, so the model ignores these values because their weights are zero. 

### Pairwise

We need to create pairwise (the relationship between AA0 AA1) relationships.... since we're comparing two AAs locations using "euclidean distance" (Pythagoras but 3D instead of 2D) as mentioned before, and each embedding is 64 dimemions.

Maybe this will stick:

In 1D sequence: [AA0, AA1, AA2], AA0 and AA2 are only two positions farther away from each other  0 - 2 = 2 positions.
BUT, in 3D space (our life but minus the "time" dimension), maybe they're neighbors. We avoided outputting 3D maps directly because of the confusion in the neural network [ * need more possibilities], so we proposed the pairwise idea of 2D dimensions. 

It goes by first adding dimensions to our first "General embedding": `emb_i` by using `. unsqueeze(zeroindexed) ` to add rows for each AA. 

First, let's see a sample data:

```python

1 protein, 3 amino acids, 2-dim embeddings (for visualization)
batch_size = 1
seq_len = 3 # Our AA's
embed_dim = 2


# "General embedding" 

embeddings = tensor([
    [[0.1, 0.2],   # AA0 embedding
     [0.3, 0.4],   # AA1 embedding
     [0.5, 0.6]]   # AA2 embedding
])

  # Shape: (1, 3, 2)


emb_i = embeddings.unsqueeze(2)


emb_i = [[[[0.1, 0.2]],   # AA0
          [[0.3, 0.4]],   # AA1
          [[0.5, 0.6]]]]  # AA2, and yes, they only have one bracket extra. 


# Original: [AA0, AA1, AA2]
# (1, 3, 2)
# After:    [[AA0], [AA1], [AA2]]  Each AA in its own "row"
#(1, 3, 1, 2)

```

Then, with another embedding, calling it `emb_j`. 

#### Why two embeddings? 

To achieve pairwise relationships. Yes, this is how we do it. 

So, the PAIRWISE embeddings should exist by adding an embedding of an AA with with another called `concat'ing`

In other words, If we had a 64 dimenional embedding for each AA, it'd be a 128 dimensional PAIRWISE embedding.

So we gt emb_j, our other pair to be concatted with the other embedding emb_i using our "General embedding".

To get emb_j, we add another dimension so all AA's can be together rather than each to be in one row


```python
# Our "General embedding"
embeddings = [[[0.1, 0.2],
               [0.3, 0.4],
               [0.5, 0.6]]]


# Original: [AA0, AA1, AA2]
# Shape: (1, 3, 2)

emb_j = embeddings.unsqueeze(1) # Here it is

emb_j = [[[[0.1, 0.2],   # All AAs in one "row" now
           [0.3, 0.4],
           [0.5, 0.6]]]]


# After:    [[AA0, AA1, AA2]]   All AAs together
# Shape: (1, 1, 3, 2)  Added dimension at position 1

```

Then we `.expand` and enter a Python confusion point out of the range of the intuition required to understand AlphaFold 1.

`emb_i = emb_i.expand(-1, -1, seq_len, -1)`

We repeat the last column three times (our` seq_length` which is AA count); 

 
```python
emb_i = [[[[0.1, 0.2],   # Focus on this
           [0.3, 0.4],
           [0.5, 0.6]]]]
```

We repeat 0.1, 0.2 by duplicating them to other columns to the right:

```python
[[0.1, 0.2], [0.1, 0.2], [0.1, 0.2]],  # Row 0: AA0
```
The same for the other AA's:

```
[[0.3, 0.4], [0.3, 0.4], [0.3, 0.4]],  # Row 1: AA1 repeated 3 times
[[0.5, 0.6], [0.5, 0.6], [0.5, 0.6]]   # Row 2: AA2 repeated 3 times
```
Which end up looking like:

```python
      Col0        Col1           Col2
Row0: [AA0]     [AA0]      [AA0]
Row1: [AA1]     [AA1]      [AA1]
Row2: [AA2]     [AA2]      [AA2]

```
Now emb_j:

`emb_j = emb_j.expand(-1, seq_len, -1, -1)`


Which is done by repeating the rows, not the columns

Visual interpretation:

```python
# Col0 Col1 Col2 
# Row0: [AA0] [AA1] [AA2] 
# Row1: [AA0] [AA1] [AA2] 
# Row2: [AA0] [AA1] [AA2]
```
What does each cell contain?

```python 
# Position (row=0, col=0):
emb_i[0, 0, 0, :] = [0.1, 0.2] # AA0
emb_j[0, 0, 0, :] = [0.1, 0.2] # AA0 # So, (AA0, AA0) pairs...

# Position (row=0, col=1):
emb_i[0, 0, 1, :] = [0.1, 0.2] # AA0
emb_j[0, 0, 1, :] = [0.3, 0.4] # AA1 # So, (AA0, AA1) pairs... etc

# Position (row=0, col=2):
emb_i[0, 0, 2, :] = [0.1, 0.2] # AA0
emb_j[0, 0, 2, :] = [0.5, 0.6] # AA2 # So, (AA0, AA2)

# Position (row=1, col=0):
emb_i[0, 1, 0, :] = [0.3, 0.4] # AA1 
emb_j[0, 1, 0, :] = [0.1, 0.2] # AA0 So, (AA1, AA0)

```

Let's concat both embeddings `emb_i` with `emb_j` each cell contains 4 dimensional vectors (2 for each AA)

Now, one may propose that we have used a different concat technique, touching on feature engineering, which is called asymmetric (where it captures direction) and symmetric order (doesn't matter) decomposition:

Symmetric + Antisymmetric decomposition 
 
 ```python
 sym = emb_i + emb_j # Symmetric
 antisym = emb_i - emb_j # Antisymmetric
```
```
Then concat them:
```
```python
pair_features = torch.cat([sym, antisym, emb_i * emb_j], dim=-1)

```
Or maybe leverage the  "sequence separation" technique, in which I have no expertise on.






Now we use convolutional layers. These are "kernels" that perform computation based on[ * ] and slide [ * ]. Note that we need reshaping to BCHW.

This is a WEIGHTED SUM of all input channels
Each output channel learns different "combinations" of input features. Then we add [[non-linearities]], in our case is a RELU activation function. This gives us depth which the problem statement deserves (Unless:
1) There are scaling techniques incorporated (more layers compensating for the lack of depth, etc...)
2) There will be a more simple algorithm (preferably comparable performance to that of the bigger AF1/2 we have now) in the future other, but that's for another topic

Now, this is my favorite second-trick: 
Remember that we had pairwise relationships? That indirectly holds us capable of sliding a window across these pairs! They truly look like boxes that can be slid over once you think about that.. 

but apparently the purpose of CONV2D was diminished after we did `kernel = 1`, which makes us unable to render neighbors and see the relationships of pairs.

Then we add residual blocks, so the gradient can flow backwards in larger architectures. 
Also to have padding ON for convnets, because having same shapes is a necessary design choice (but not strictly) for the residual connection to work.

After passing  through the network's head, it outputs 37 logits as bin ranges output that we later convert with softmax 

Then we initialize specific parts of the network as demonstrated here, by simply looping on the list of modules that we have (Architecture components like CONV/). This aids in deep NN architecture or if we needed to scale.

### Inference and recap (After engineering)

Given a novel, non-seen-before protein sequence: `MSVTQRFIAKQ`(1)
We generate a MSA (2) using an external DB (HHblits), then generate a PSI-blast profile. Then, after the output of the distance map from the NN, using the distance map, Then use MDS or similar tool to construct a 3D, folded protein. We optionally can compute the Angle Matrices using DSSP analysis to label H/E/C on the SS per AA.

Therefore, we ingest (1), (2) and (3) to the model, with it predicting without "improving" thus not using Cα coordinates (unless we specifically add TrainingTestTime functionality), because we're inferencing after all. No gradients, weight updates, activations.

Note that AF1 is far away from being an E2E architecture, because of the external MSA generation and requiring GradientDecsent (GD) optimization rather than directly predicting coordinates.

[Continue]
## To be added:

- Linking factual information and separating it from speculative theory
- Including the original paper in attempt of paper replication, not truly word-for-word but close enough
- Align the topics to their respective headings. Some are scattered!
- Many other images for demonstration and illustration. Maybe a self-made GIF to clear up both the bigger and smaller pictures. Heavy intermediate factors may be distracting an

Important links:

Citation: Senior, A.W., Evans, R., Jumper, J. et al. "Improved protein structure prediction using potentials from deep learning." _Nature_ 577, 706–710 (2020).

https://pmc.ncbi.nlm.nih.gov/articles/PMC11348012/

https://medium.com/@satishlokhande5674/what-information-does-alphafold-use-from-the-protein-data-bank-44048e672100

https://github.com/google-deepmind/deepmind-research/tree/master

