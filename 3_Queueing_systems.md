# Queueing Systems

## BCp3 – PBS

## BCp4 – SLURM

## Environment modules and queueing systems

One important difference between PBS and SLURM is how environment is preserved when you run a job:

- For **PBS**, _the job runs in a clean environment_, so any modules you load or variables you set in your interactive session on the login node will not be automatically forwarded to compute jobs. However, your shell start-up scripts will still be executed, e.g. commands in `.bashrc` will be run.
- For **SLURM**, _jobs start in the same environment that you had on the login node_, so you don't _need to_ load all the required module in your job script.

Regardless of which system you're using, it is good practice to only have loaded those modules that are required for your job.
This avoid issues where your job is affected by modules you may have loaded for testing (or other purpuses) and forgot about.
Therefore, consider starting your job script with a `module purge`, followed by `load`s for _only the required modules_.
