# Lab 08: Knowledge check

**Answer each question below it, in your own words, in one to three sentences.** Replace the `...` below **Your answer:** with your text. Questions 1 and 3 also ask for saved output, which goes into the directory `results/`.

## 1. Slurm: what jobs have I submitted?

**Your task:** print a history of all your Slurm job submissions *since November 4, 2026*. Format the output to include the job ID, job name, node list, partition, account, user, state, start time, and elapsed time. See the job accounting fields in the [sacct documentation](https://slurm.schedmd.com/sacct.html). You may also ask Claude Code or another LLM for the exact command. 

Redirect the command's output to `results/job-history.txt` and include the file in your submission. Do not worry if some earlier jobs failed or were canceled. The complete history should show your process, including trial and error.

*Hint:* See [Part 1, Step 6](README.md#step-6-check-how-the-tasks-ended) in the README.

**Your answer:** Provide the entire command you used, including all CLI options.

sacct -X --starttime=2026-11-04 --format=JobID,JobName,NodeList,Partition,Account,User,State,Start,Elapsed > results/job-history.txt

## 2. Slurm: did every array task run at the same time?

You submitted your entire job array with one `sbatch` command, but Slurm scheduled each task separately.

**Your task:** in the job history from Question 1, find the tasks from your most recent completed array run. Compare their start times and elapsed times.

*Hint:* See [Part 1, Step 5](README.md#step-5-watch-the-job-array) in the README.

**Your answer:** Did all tasks in the array start at the same time? Was the elapsed time identical for every task? Explain why or why not.

No, the tasks did not all start at the same time: the first ones started at 16:37:57 and the last ones at 16:58:15, in groups of about 8, with each new group starting once the previous group finished and freed up GPU slices. This happens because Slurm schedules each task separately and only starts one when a GPU slice is available, and the slices on gpu-mig are shared with everyone on the cluster. The elapsed times were also not identical (15 to 21 seconds), because each task processed a different book and other jobs were running on the same node at the same time.

## 3. Slurm: did my job use what it asked for?

Suppose your next job array has 1,000 books instead of 101. Before you submit it, you want to know whether your script asks for too much or too little memory and time.

**Your task:** run `seff` for **one** task of your finished job array, and save its output as `results/seff.txt`.

*Hint:* `seff` takes the ID of one task, such as `4211873_1`. Compare its **Memory Utilized** and **Job Wall-clock time** with `--mem` and `--time` in your `summarize.slurm`.

**Your answer:** How much memory and time did the task use, compared with what your script asked for? What would you change in your script, and why?

The task used 1.37 GB of memory, about 9% of the 16 GB requested, and ran for 18 seconds out of the 5-minute time limit. For the next job, I would lower --mem to about 5G and --time to about 1 minute, because smaller requests are easier for Slurm to fit, so tasks can start sooner, and they leave more resources for other people’s jobs. I would still keep some extra room, because some books are longer than others (my tasks took up to 21 seconds) and Slurm kills any task that goes over its memory or time limit.
