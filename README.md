Create an environment and install the required dependencies with:

```bash
conda create --NEURO_PROJECT1 env python=3.10
conda activate NEURO_PROJECT1
pip install -r requirements.txt
```

Add a folder data/ in which you put "AO026_20181122_180943.nwb", the data provided to us in this drive: https://drive.google.com/drive/u/1/folders/0AH5XZOjakvTtUk9PVA 

IMPORTANT: Check that these dara are ignored via .gitignore, the file is too big to be pushed on GitHub.

nwb_filter.ipynb is the notebook where we filter the data to keep only the one we need.

experiment.ipynb is the notebook where we run the experiment and we test wether our code vector is good or not in order to distinguish stim_trial and no_stim_trial.

The final goal of this project is to use this code as input for a simulation of a reward-base learning experiment.