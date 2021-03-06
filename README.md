# Gophish Python CLI

This tool aim to perform huge phishing campaigns by using the very respected gophish toolkit. If you need to run a campaign for more than 10 000 email addresses or need to split a batch of email addresses into smaller groups for any reasons (lower risks with anti-Spam, avoid being banned by IPS, bypass Email service limitations, etc.), that tool could help you!

The tool is based the [api-client-python](https://github.com/gophish/api-client-python) library.

A reporting feature was recently added (2018-03-10) allowing users to generate statistics for your report. You can configure the tool to download apache logs, access sendgrid stats (including bounces, deffered, spam reports, etc.), access empire agents and correlate other useful data.


## Installation

To begin, you need a gophish instance. Find more about the gophish project [here](https://github.com/gophish/gophish).

To install `gophish-cli`, simply run the command:

```bash
git clone --recursive https://github.com/gosecure/gophish-cli
cd gophish-cli/api-client-python
pip3 install -r requirements.txt --user
python3 ./setup.py install --user
cd ..
pip3 install -r requirements.txt --user
```

To install `gophish-cli` in a virtualenv, run:

```bash
git clone --recursive https://github.com/gosecure/gophish-cli

virtualenv ./gophish-cli
cd gophish-cli
source bin/activate

cd api-client-python
pip3 install -r requirements.txt
python3 ./setup.py install
cd ..
pip3 install -r requirements.txt
```

To test harvested credentials on OWA, you will also need to install the [exchangelib](https://github.com/ecederstrand/exchangelib/) submodule.

```bash
cd exchangelib
python3 ./setup.py install --user
```

It is currently using a forked version of the [API library](https://github.com/gosecure/api-client-python/) for development purpose.


## Configuration

You need the API key found in the [Settings page](https://gophish.gitbooks.io/user-guide/content/documentation/changing_user_settings.html#changing-your-password--updating-settings).

Then run `cp config.default.py config.py` and edit the `config.py` file using your favourite text editor.

### Step 1 - Connection to the gophish instance

```python
API_KEY = ''
API_URL = 'http://127.0.0.1:3333
```


### Step 2 - Define the campaign parameters

As mentionned earlier, `gophish-cli` true power is for campaign spliting. Thus, the objects below must be created using the webUI:

 * Landing Page
 * Email Template
 * Sending Profile

The tool will take care of the email groups and campaigns creation. It will also let you retrieve statistics and credentials from the same batch.

The three parameters below should be configured based on the number of email addresses:

 * GROUP_SIZE: Number of email addresses per group. 
 * START_INTERVAL: Interval before starting the first batch.
 * BATCH_INTERVAL: Interval between each batches.


### Step 3 - Spam!

The most automated way to run the tool is as follow:

```bash
$ python3 ./gophish-cli.py campaign --start --new-groups
[-] Preparing new groups creation.
[-]   Campaign Name: JohnDoe
[-]   File Path: /path/to/test_emails.txt
[-]   Batch size: 30
[-]   Group count: 4
Do you want to continue? [y/N] y
[-] Creating group "JohnDoe - Group 1" with 30 targets. First email is johndoe1@trash-mail.com
[-] Creating group "JohnDoe - Group 2" with 30 targets. First email is johndoe31@trash-mail.com
[-] Creating group "JohnDoe - Group 3" with 30 targets. First email is johndoe61@trash-mail.com
[-] Creating group "JohnDoe - Group 4" with 10 targets. First email is johndoe91@trash-mail.com
[-] Preparing to launch campaigns
[-]   Campaign Name: JohnDoe
[-]   Landing Page: LP - EN - aCampaign - JohnDoe
[-]   Email Template: ET - EN - aCampaign - JohnDoe
[-]   Sending Profile: imgonahackyou.com (provider X)
[-]   URL: https://johndoe.imgonahackyou.com
[-]   Group count: 4
[-]   Launch Date: 2017-02-06 17:54:46.813515-05:00
[-]   Time interval: 1 minute(s)
Do you want to continue? [y/N] y
[-] Launching campaign "JohnDoe - Group 1" at 2017-02-06 17:54:46.813515-05:00
[-] Launching campaign "JohnDoe - Group 2" at 2017-02-06 17:55:46.813515-05:00
[-] Launching campaign "JohnDoe - Group 3" at 2017-02-06 17:56:46.813515-05:00
[-] Launching campaign "JohnDoe - Group 4" at 2017-02-06 17:57:46.813515-05:00
```


## Help

```
$ python3 ./gophish-cli.py -h               
usage: gophish-cli.py [-h] [-v] [-c CONFIG] [-d]
                      {group,campaign,creds,stats,report} ...

Gophish cli. Use this tool to quickly setup a phishing campaign using your
gophish infrastructure.

positional arguments:
  {group,campaign,creds,stats,report}
    group               Manage groups.
    campaign            Manage campaigns.
    creds               Manage credentials.
    stats               Manage statss.
    report              Generate report (currently multiple CSV files).

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
  -c CONFIG, --config CONFIG
                        Alternative config file. Default is config.py (Not
                        implemented yet)
  -d, --debug           Run the tool in debug mode
```

Every positional arguments have its own help page. For example: `./gophish-cli.py campaign -h`. Read them for more details. 


## Post-campaign useful commands

To get results

```
$ python3 ./gophish-cli.py campaign --results
[-] Exported 2492 timeline entries to /some/path/campaign_results_JohnDoe.csv
[-] Exported 40 credentials to /some/path/campaign_creds_JohnDoe.csv.
```

To print credentials

```
$ python3 ./gophish-cli.py creds --print
+---------------------+--------------+------------+
| Email               | User         | Pass       |
+---------------------+--------------+------------+
| mdube@gosecure.ca   | mdube        | P@$$w0rd1! |
| somebody@gouv.qc.ca | lddoei       | Winter2017 |
| ...                 | ...          | ...        |
+---------------------+--------------+------------+
```


To test credentials by using OWA

```
$ python3 ./gophish-cli.py creds --test-owa
[-] **WARNING**
[-] Too many attempts could lock accounts. Be easy with this feature.
[-]
[-] Preparing to test credentials on OWA
[-]   Campaign Name: Mart
[-]   OWA Domain: LAB
[-]   OWA Server: owa.lab.local
[-]   Credentials count: 123
Do you want to continue? [y/N] y
LAB\user1 - Fall2008: Successful login
LAB\user2 - Milhouse44$: Failed login
...
```


To get source IP addresses

```
$ python3 ./gophish-cli.py stats --targets-ip

+------------------------+-----------+
| IP Address             | Hit Count |
+------------------------+-----------+
| No IP. Email Sent Only |    1251   |
| 31.10.39.30            |     50    |
| 2.21.14.65             |     2     |
| ...                    |    ...    |
+------------------------+-----------+
```

## Reporting

Phishing is fun but often comes with a reporting phase. Once configured, the reporting feature is a great kickstarter for your report.

```
$ ./gophish-cli.py -d report

DEBUG    Arguments: Namespace(action='report', config=None, debug=True)
INFO     Getting 1 campaign timelines for Client X
DEBUG      Got 7605 events
INFO     Getting 1 campaign results for Client X
DEBUG      Got 4820 events
INFO     Generating report.
INFO     Setting up folders
DEBUG    Creating folder: /home/user/client/phishing/report_20180310-133544/
DEBUG    Creating folder: /home/user/client/phishing/report_20180310-133544/apache_logs/
INFO     Downloading apache logs
DEBUG    [SSH] Attempting to authenticate
DEBUG    [SSH] Trying ssh-agent key <some fingerprint>
DEBUG    [SSH]... success!
DEBUG    [SSH] Authentication successful
DEBUG    [SSH] Changing directory: /var/log/apache2/
DEBUG    [SSH] Downloading: meh.domain.com.access.log
DEBUG    [SSH] Downloading: meh.domain.com.access.log.1
DEBUG    [SSH] Downloading: meh.domain.com.error.log
DEBUG      Got 119 lines
INFO     Getting Sendgrid Stats
INFO     Getting Empire Agents
DEBUG      Got 82 agents
INFO     Extracting stats
INFO     Printing Report
Raw Data: 

  Timeline: 
    First Event: 2018-03-09 08:09:46
    Last Event: 2018-03-10 09:20:15
    Email sent: 6165
    Email opened: 1052
    Clicked Link: 238
    Submitted Data: 149
    Unique Email opened: 188
    Unique Clicked Link: 158
    Unique Submitted Data: 103
    Source IPs: 
      None (6166)
      1.2.3.4 (800)
      5.6.7.8 (225)
      ...

  Sendgrid stats:
    Blocks: 13
    Bounce Drops: 188
    Bounces: 98
    Clicks: 0
    Deffered: 10
    Delivered: 5101
    Invalid Emails: 0
    Open: 1227
    Processed: 6014
    Requests: 6068
    Spam Report Drops: 0
    Spam Reports: 0
    Unique Clicks: 0
    Unique Opens: 193
    Subscribe Drops: 0
    Unsubscribes: 0

  Apache: 
    Malware Download: 149
    Source IPs: 
      1.2.3.4 (123)
      5.6.7.8 (45)

  Empire: 
    Agents count: 82
    Agents HighPriv count: 10
    Unique Agents username count: 78
    Unique Agents Hostnames count: 76
    OS Details: 
      Microsoft Windows 7 Professional  (75)
      Microsoft Windows 10 Enterprise (4)
      Microsoft Windows 10 Home (1)
      Microsoft Windows 7 Ultimate  (1)
      Microsoft Windows 10 Pro (1)
    Source IPs: 
      1.2.3.4 (40)
      5.6.7.8 (10)
      ...

Analyzed Data: 

  Conversion Percentage:
    Email Received (6165) -> Email Opened (1052): 17.06
    Email Open (1052) -> Link Clicked (158): 15.02
    Page Visit (158) -> Send Credentials (103): 65.19
    Malware Download (149) -> Malware Execution (78) (Empire): 52.35
    Malware Download (149) -> Malware Execution (0) (Msf): 0.0
    Malware Download (149) -> Malware Execution (0) (Cobalt): 0.0
```

## Known issues & Troubleshooting

### Issues with Outlook 365

Outlook 365 limit the number of email sent per connection to 30. `GROUP_SIZE` must be set to 30 when using this provider.

### Emails keep the status "Sending"

On previous version of gophish (< 0.5), some email addresses were stuck with status "Sending" if too many emails were sent at once. The fix was to split in groups of 100 and put a delay between each waves. 

Example: `GROUP_SIZE=100` and `BATCH_INTERVAL=5`

### My emails are flagged as SPAM.

Test your campaign here: [https://www.mail-tester.com/](https://www.mail-tester.com/)
