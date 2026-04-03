At its core, the `Metasploit Project` is a collection of commonly used tools that provide a complete environment for penetration testing and exploit development.

The `modules` mentioned are actual exploit proof-of-concepts that have already been developed and tested in the wild and integrated within the framework to provide pentesters with ease of access to different attack vectors for different platforms and services. 

Metasploit is not a jack of all trades but a swiss army knife with just enough tools to get us through the most common unpatched vulnerabilities.

---

## MSFConsole

It provides an "all-in-one" centralized console and allows you efficient access to virtually all options available in the `MSF`.

The features that `msfconsole` generally brings are the following:

- It is the only supported way to access most of the features within `Metasploit`
- Provides a console-based interface to the `Framework`
- Contains the most features and is the most stable `MSF` interface
- Full readline support, tabbing, and command completion
- Execution of external commands in `msfconsole`


## Understanding the Architecture

To fully operate whatever tool we are using, we must first look under its hood. It is good practice, and it can offer us better insight into what will be going on during our security assessments when that tool comes into play. It is essential not to have [any wildcards that might leave you or your client exposed to data breaches](https://www.cobaltstrike.com/blog/cobalt-strike-rce-active-exploitation-reported).

By default, all the base files related to Metasploit Framework can be found under `/usr/share/metasploit-framework` in our `ParrotOS Security` distro.

#### Data, Documentation, Lib

These are the base files for the Framework. The Data and Lib are the functioning parts of the msfconsole interface, while the Documentation folder contains all the technical details about the project.

#### Modules

The Modules detailed above are split into separate categories in this folder. We will go into detail about these in the next sections. They are contained in the following folders:

        shellsession
`hellopriyanshu2702@htb[/htb]$ ls /usr/share/metasploit-framework/modules auxiliary  encoders  evasion  exploits  nops  payloads  post`

#### Plugins

Plugins offer the pentester more flexibility when using the `msfconsole` since they can easily be manually or automatically loaded as needed to provide extra functionality and automation during our assessment.

        shellsession
`hellopriyanshu2702@htb[/htb]$ ls /usr/share/metasploit-framework/plugins/ aggregator.rb      ips_filter.rb  openvas.rb           sounds.rb alias.rb           komand.rb      pcap_log.rb          sqlmap.rb auto_add_route.rb  lab.rb         request.rb           thread.rb beholder.rb        libnotify.rb   rssfeed.rb           token_adduser.rb db_credcollect.rb  msfd.rb        sample.rb            token_hunter.rb db_tracker.rb      msgrpc.rb      session_notifier.rb  wiki.rb event_tester.rb    nessus.rb      session_tagger.rb    wmap.rb ffautoregen.rb     nexpose.rb     socket_logger.rb`

#### Scripts

Meterpreter functionality and other useful scripts.

        shellsession
`hellopriyanshu2702@htb[/htb]$ ls /usr/share/metasploit-framework/scripts/ meterpreter  ps  resource  shell`

#### Tools

Command-line utilities that can be called directly from the `msfconsole` menu.

        shellsession
`hellopriyanshu2702@htb[/htb]$ ls /usr/share/metasploit-framework/tools/ context  docs     hardware  modules   payloads dev      exploit  memdump   password  recon`

