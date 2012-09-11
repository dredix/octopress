---
layout: post
title: "Dynamic DNS Updates on Namecheap Via a PowerShell Script"
date: 2012-09-10 23:38
comments: true
categories: 
 - PowerShell
 - Web hosting
 - Dynamic DNS
---
Last year I decided to transfer my domains from GoDaddy to [Namecheap](http://www.namecheap.com/), for [no](http://geekfeminism.wikia.com/wiki/Go_Daddy's_advertising) [particular](http://www.huffingtonpost.com/2011/03/31/bob-parsons-godaddy-ceo-elephant-hunt_n_843121.html) [reason](http://arstechnica.com/tech-policy/2011/12/godaddy-faces-december-29-boycott-over-sopa-support/). With the new registrar I wanted to try some things I never got around to do before, including setting up a subdomain for easy remote access to a location I shall henceforth refer to as Home.

Since Home lacks a static IP I had to enable dynamic DNS for the domain, which basically allows changing the IP address associated to a given subdomain programmatically and with immediate effect. Fortunately there is [plenty of information](http://www.namecheap.com/support/knowledgebase/category.aspx/11/dynamic-dns) on [what needs to be done](http://www.namecheap.com/support/knowledgebase/article.aspx/36/11/what-should-i-do-in-order-to-start-using-dynamic-dns).

The step I had trouble with was finding a client to perform the DNS updates on a regular fashion, say, every hour, from a Windows PC behind a proxy that requires authentication. Now Namecheap has a [Dynamic DNS client of their own](http://www.namecheap.com/support/knowledgebase/article.aspx/111/11/using-namecheap-dynamic-dns-client-version-20x-beta), but for reasons I can't remember at this moment it didn't work for me when I tried it. Namecheap also provides a list of [alternate clients](http://www.namecheap.com/support/knowledgebase/article.aspx/5/11/are-there-any-alternate-dynamic-dns-clients), but again none of them fitted my bill exactly. As a last resort, they also allow updating the DNS [via a HTTP GET request](http://www.namecheap.com/support/knowledgebase/article.aspx/29/11/how-to-use-the-browser-to-dynamically-update-hosts-ip), which means you can easily do it with a web browser or programmatically with an appropriate HTTP command/library.

So even though this is probably a problem that has been solved many times, I wanted to indulge in a little bit of [NIH](http://en.wikipedia.org/wiki/Not_invented_here) and write my own client as a PowerShell script. Specifically I wanted to:

1. Load parameters from a configuration file
2. Retrieve current public IP address
3. Compare IP address against the one on Subdomain's A record
4. Update A record if found different
5. Send an email with the results

Here's is what I did.

### Configuration File

Instead of hardcoding things like the domain and password I wanted to load configuration details from an [INI file](http://en.wikipedia.org/wiki/INI_file), so that the script could be reusable. Why INI and not, say, XML? Because for such a simple list of parameters I find the INI format much easier to read. I could also have used [YAML](http://www.yaml.org/), but then it's just a simple change from equal sign to colon on the parser. Anyway, here's the full list of parameters to load from the INI file:

``` ruby
# The subdomain to be updated, which has to be previously configured 
# as an A record.
DDNSSubdomain   = "subdomain"
# The domain registered through Namecheap, with Dynamic DNS enabled.
DDNSDomain      = "mydomain.com"
# The password for authorising the updates, obtained from Namecheap
# upon enabling Dynamic DNS (see http://tinyurl.com/d6ao9de).
DDNSPassword    = "1234567890abcdef1234567890abcdef"
# An email account from which a message with the results of the
# update attempt will be sent.
EmailFrom       = "from@example.com"
# The recipient of the message.
EmailTo         = "to@example.com"
# The SMTP server for sending the email. I'm currently using GMail.
SMTPServer      = "smtp.gmail.com"
# The SMTP port.
SMTPPort        = 587
# SMTP username. If you are sending messages through GMail, then it
# would be your Google account.
SMTPAuthUsername= "gmailuser"
# SMTP password.
SMTPAuthPassword= "p4ssw0rd"
# If connecting through a proxy, this parameter should be set to 1.
# Otherwise it should be 0 or left unitialised
ProxyEnabled    = 1
# The proxy address. Ignored if ProxyEnabled is not 1.
ProxyAddress    = "proxy.mydomain.com"
# The proxy port.
ProxyPort       = 8080
# Proxy domain.
ProxyDomain     = "corporate"
# Proxy credentials, if required.
ProxyUser       = "domainuser"
# See above.
ProxyPassword   = "domainpassword"
```

For parsing the file I used the following snippet, adapted from [Artem Tikhomirov's code posted on Stack Overflow](http://stackoverflow.com/a/422529):

``` ruby
Function Parse-IniFile ($file) {
    $ini = @{}
    switch -regex -file $file {
        "^\s*([^#].+?)\s*=\s*\`"*(.*?)\`"*$" {
            $name,$value = $matches[1..2]
            $ini[$name] = $value.Trim()
        }
    }
    $ini
}
```

### Retrieving current public IP address

With [DNS-O-Matic](http://myip.dnsomatic.com) you can get your current public IP address in a plain format so there is nothing to parse and the result can be used immediately. I don't remember where I got the tip from so I'll just give [this guy the credit](http://poshcode.com/2661):

``` ruby
# Retrieve public IP address
$CurrentIp = (New-Object System.Net.WebClient).DownloadString('http://myip.dnsomatic.com/')
# Validate using a regular expression (only IPv4 supported at this stage)
$Pattern   = '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}'
if (!($CurrentIp -match $Pattern)) {
    Log-Error "A valid public IP address could not be retrieved"
    # Error handling
}
```

### Comparing IP addresses and updating A record 

I need to perform the GET request to update the A record, but only if the current public IP address is different to the one already set. I do not want, however, to spam the Namecheap servers with continuous queries every time I want to verify the existing value. So instead of retrieving the A record from Namecheap, I use a local environment variable to store the last known IP address. That works because I'm updating the IP from a single server, so there are no possible conflicts to be worried about.

``` ruby
# Retrieve IP address stored on environment variable.
$StoredIp  = [Environment]::GetEnvironmentVariable("PUBLIC_IP","User")

# If IP address hasn't changed, then finish the script.
if ($StoredIp -eq $CurrentIp ) {
    # Log message, etc.
    exit 0
}

# Otherwise, create update URI and perform a Web request.
$UpdateUrl     = "https://dynamicdns.park-your-domain.com/update?host=$DDNSSubdomain&amp;domain=$DDNSDomain&amp;password=$DDNSPassword&amp;ip=$CurrentIp"
$UpdateDDNS    = (New-Object System.Net.WebClient).DownloadString($UpdateUrl)

# Error handling omitted for clarity. Or laziness.

# Finally, set the environment variable to the new IP address
[Environment]::SetEnvironmentVariable("PUBLIC_IP", $CurrentIp, "User")
```

### Sending an email to notify the change

Next is to report the change of IP address via email, so I can validate and/or troubleshoot in case something went wrong.

You can choose between writing a log to a local text file, for example, and then include that file as an email attachment. In my case, since I never expect the logs to be too large, I'm just concatenating to a string in memory and then writing to the body of the message. Here's my function for sending the email, based on [this code from Christian Muggli](http://stackoverflow.com/a/2250309/1014): 

``` ruby
# Send an email with the contents of the log buffer.
# SMTP configuration and credentials are in the configuration dictionary.
function Email-Log ($config, $message) {
    $EmailFrom        = $config["EmailFrom"]
    $EmailTo          = $config["EmailTo"]
    $EmailSubject     = "DDNS log $(get-date -format u)"  
      
    $SMTPServer       = $config["SMTPServer"]
    $SMTPPort         = $config["SMTPPort"]
    $SMTPAuthUsername = $config["SMTPAuthUsername"]
    $SMTPAuthPassword = $config["SMTPAuthPassword"]

    $mailmessage = New-Object System.Net.Mail.MailMessage 
    $mailmessage.From = $EmailFrom
    $mailmessage.To.Add($EmailTo)
    $mailmessage.Subject = $EmailSubject
    $mailmessage.Body = $message

    $SMTPClient = New-Object Net.Mail.SmtpClient($SmtpServer, $SMTPPort) 
    $SMTPClient.EnableSsl = $true 
    $SMTPClient.Credentials = New-Object System.Net.NetworkCredential("$SMTPAuthUsername", "$SMTPAuthPassword") 
    $SMTPClient.Send($mailmessage)
}
```

[The full script can be downloaded from Github](https://gist.github.com/3508143).

To run the query, open a console and execute the following line:

	powershell -command "& 'C:\path\to\script\UpdateDDNS.ps1 -c config.ini' "


The final step would be to schedule the command so it runs every hour, for example. You just need to set up a task on Windows Task Scheduler for that, but since you asked, [here are some instructions to get you started](http://blogs.technet.com/b/heyscriptingguy/archive/2012/08/11/weekend-scripter-use-the-windows-task-scheduler-to-run-a-windows-powershell-script.aspx).

So that's it: A quick, cheap and easy way to keep your Namecheap domains with Dynamic DNS up to date. Now that we're done, let's return to bashing Bob Parsons on Twitter, shall we?

