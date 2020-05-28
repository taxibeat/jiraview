# JIRAVIEW

Can your JIRA instance do this?
![fancy plot](plots/days-in-status.png "Fancy plot")

Or this?
![othe fancy plot](plots/comments-vs-resolution-days.png "Other fancy plot")

Neither can mine. That's why I have this project. It extracts the raw data from any JIRA instance (that has REST API support) and creates useful data files from it.

## How it works
The scripts talk to JIRA via the REST API and extract all data for issues that match a given [JQL](https://confluence.atlassian.com/display/JIRA/Advanced+Searching "JIRA advanced searching") query. The extracted data is inserted into a MongoDB instance. There is support for incremental syncing from JIRA to MongoDB. The data in MongoDB can be saved into several different types of files (see below).

An intermediate data store (MongoDB) is used because JIRA has relatively high response times and we want to be able to use all issue data at any time.

## What's in the box

There are several things in here.

### Python
The python folder contains the core scripts that perform data extraction and transformation.

### Ansible
There is an Ansible playbook that can take a bare CentOS installation and install all dependencies required for running the jiraview scripts (i.e. Python + MongoDB), R and Jenkins, which we often use for scheduling things. With this setup you can create a machine that runs the JIRA extraction and R scripts to generate plots on a certain schedule.

### R

An example R script that generates a number of (not necessarily) useful plots from the CSV files.

## Using

(Assumes prior knowledge of python virtual environments and the virtualenvwrapper.)

In your favorite shell, take these steps:

	# Make sure you are running MongoDB* on localhost without authentication required
	# The database name that will be used is 'jiraview'
	
	git clone git@github.com:taxibeat/jiraview.git
	cd jiraview
	mkvirtualenv jiraview	
	cd python
	
	pip install -r requirements.txt
	cd jiraview
	
	# Now we define a data set (for the Apache Wicket project)
	# Note tha {last_update} placeholder in the JQL query. This is required for incremental fetching.
	python dataset.py wicket \
	-jql 'project = Wicket and updatedDate > {last_update}' \
	-url 'https://issues.apache.org/jira/' \
	-collection wicket
	
	# Now we fetch all issues
	# -v is for verbose; if ommitted, you won't see any output
	# This will take a long time; if you run the fetch script again, it will do an incremental fetch
	python fetch.py wicket -v
	
	# Create a bunch of CSV files from the issues
	# (see the R sample script for usage; column names in CSV's are reasonably descriptive)
	python extract.py wicket -dir /tmp
	
	# You can now use the CSVs for your JIRA data needs.

Other available scripts are:

- jsondump.py, which will output a JSON file with all issue data (one JSON object per line; see the [JIRA REST API docs](https://docs.atlassian.com/jira/REST/latest/) for what's in there)
- xes.py, which create a XES XML file that can be used in the [ProM open source process mining tool](http://www.processmining.org/prom/start).

Use the -h option on each script to find all command line parameters.

### Installing MongoDB locally on MacOS

```
	brew tap mongodb/brew
	brew install mongodb-community

	# And starting it (also possible to start as a daemon)
	mongod --config /usr/local/etc/mongod.conf
```

### Fetch & dump JIRA issues to a JSON

Typically, you want to configure a dataset query, then fetch all related issues and dump data to JSON file:

``` 	
 	python dataset.py -jql 'project in ("<Your-project-id-1>", "<Your-project-id-2>") AND updated >= last_update' -collection dataset -user <Your-Jira-user-id> -password <Your-Jira-password> -url https://jira.<your-company>.com/ <dataset-name>
 
 	python fetch.py -v <dataset-name>
 
 	python jsondump.py -basename <file-base-name> -dir . <dataset-name>
```