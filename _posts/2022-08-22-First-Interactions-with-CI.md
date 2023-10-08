---
layout: post
author: jack
category: [software,hardware]
--- 

My First interaction with the concept of CI (regression) in the world of chip design

As an Engineer, my job involves bringing in both software and hardware/chip worlds together. Similar to the software development, continuous development happens to the design/IP and it needs to be tested for both PPA and tool compatability. Though I was first involved in developing a regression workflow for opensource physical verification EDA tools like Magic and Netgen, it was more like automating the process of running designs, extracting reports, parsing them and checking if any errors are generated. I developed this workflow in 2021 and named it as Vezzal (The testing was happening more inside a docker container and the word &#39;Vessel&#39; - a controlled environment for testing - came into my mind and I just went with it). As of 2023, this is still checking both the tools with the testcases. This was my first encounter to the concept of regression where I used docker and github actions.

Later I was involved in a few projects where I started developing regression flows using Jenkins for workflows, python for intermediate automation etc. But OpenFASoC CI is a bit complicated given the generators uniqueness and the list of parameters that needs to be analyzed. I will try to give an overview of this in this post. 
<h3>How OpenFASoC CI works</h3>

I would categorize the CI setup into three types based - tool update compatability, performance check and dataset development for model building. I will try to explain all three workflows, their way of implementation and the challenges involved in them.
<ul>
<li><strong>Tool update compatability</strong> - 
Whenever a dependency is updated, generators need to be tested with the latest tools. Here first a docker image is built with the latest version of tools installed in it and the container is started on the github runner. Next the OpenFASoC repo is cloned and a generator is run to see if the generator is able to generate end gds files without any problem. This is more like a flow-flush. Once this step is completed successfully, the versions number are extracted from the docker container and stores in a file which is later pushed back to the same repository via a PR using a bot account named &quot;openfasoc-bot&quot;. If there are no changes, no push is made. The bot account creates a PR to add the changes to the main repo. This entire process is done every day (its a cron job). 
Here is the link to the workflow - <a href="https://github.com/idea-fasoc/OpenFASOC/blob/main/.github/workflows/verify_latest_tools_versions.yml">https://github.com/idea-fasoc/OpenFASOC/blob/main/.github/workflows/verify_latest_tools_versions.yml</a></li>
</ul>
Now to this method, instead of going just for a flow flush of the generator, the performance of the generator can be calculated and checked if the tool version is improving or reducing it. Currently only a single generator is tested, but all the &#39;active&#39; generators can be tested for the same.
<ul>
<li><p><strong>Performance Check</strong> - 
This is more for the generator itself. When the generator is updated or, more importantly, when a PR related to a particular generator is created, the performance of that generator based on the PR changes (not on the main repository) need to be checked. Now a generator can be launched in various configurations so either the performance need to be checked for all possible configurations or a few selected ones (time and compute optimized). Currently (as of June 2023), no such check is performed and in place of that a flow flush is done on all generators (yes all generators are checked because there are a few scripts and PDK related files that are common to all generators) to check if the updates are breaking any of the generators. However work has started to extend the current flow and add the performance metrics to a MongoDB for records. The challenge here is to check for all possible configurations which will take a huge amount of time and computation power. Also running for all possible configurations everytime an update is commited to the repo is not a smart solution either. Hence this type of workflow needs to be triggerred either manually or by a specific commit message. </p>
</li>
<li><p><strong>Dataset generation</strong> - 
This is a advance version of the above workflow. Here the generator needs to be tested for all possible configurations and the respective metrics (should be defined) should be extracted and stored in the DB for building predictive models. This is done because running a generator for a random configuration (that might not be in the dataset), dealing with a heavy run time and compute load issues, and not getting satisfied with the metrics in the end is not a smart way to use these generators. The major challenge here is the time take and the computation power especially for the simulations. Now if all simulations are launched in parallel (which are cross the number even more than 2000), the VM will crash because of the insufficient memory. The trick here is to launch them by splitting them by some means and running each split on an individual VM. The data transfer from and to VM also needs to taken care. Hence this is the complicated workflow among the three. 
Recently I was introduced to a GCP product named Batch (<a href="https://cloud.google.com/batch">https://cloud.google.com/batch</a>) which will do all the above jobs automatically - sharing of load, transfer of data, creating/managing/destroying the VMs etc. This method is currently being tested but I see this as a more promisible solution rather than using a infrastructure management tools suite like Terraform, Kubernetes etc.</p>
</li>
</ul>
<h3>Conclusion</h3>
<p>I think this is really interesting because the traditional way of CI is fully automated whereas here each step should be custom developed (although there are possibilities where things can be automated like that of software model).</p>