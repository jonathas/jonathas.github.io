---
layout: post
title: "Experiencing 10 Programming Languages"
excerpt: "I never understand when some people defend one programming language or full stack and never wanna know about other technologies and learn more about their differences"
tags: [apache, csharp, groovy, java, javascript, lua, perl, php, python, ruby, top10, dotnet]
comments: true
date: 2015-03-21T18:02:27-03:00
---

I never understand when some people defend one programming language or full stack and never wanna know about other technologies and learn more about their differences. You see, a programming language is just a tool. 

During my career, I've met many people who would be very faithful to one kind of technology and ignore all the rest. Don't get me wrong, of course we should become specialists in one kind of technology. But for example, if you stick to PHP only, you'll be missing how beautiful Rails is, or if you stick to the Microsoft stack, you'll be missing so many other technologies that don't depend on the newest version of OS or IDE to work properly and in their newest version. 

Technology is evolving all the time and it's hard to keep up with all the trends that keep coming, so I believe we must be always willing to learn more and grow in the process. After having to learn .NET to develop some systems for the company where I work, coming from my mostly-PHP background, I started to believe that given enough time, anything is possible. 

Recently I decided to get a task given to me in an interview for a Software Engineer position at Google and explore some programming languages with that idea. I thought about developing the same code with 10 different languages, just to practice, because I believe that's the best way to learn a language for me. 

I learn much better by doing (and practicing Google-fu, of course). The idea was to read an imaginary log file with a list of IP addresses and return the TOP 10 most repeated ones, along with their count. I decided then to use a real log file from a random day for that and parse it using the languages I chose for the task, which were **C++**, **C#**, **Groovy**, **Java**, **JavaScript**, **Lua**, **Perl**, **PHP**, **Python** and **Ruby**. 

The ones that I had more fun with were:

### Ruby
{% highlight ruby %}
def read_parse(filepath)
	ip_count = {}

	File.open(filepath, "r") do |f|
	  f.each_line do |line|
	    
	    ip_address = line.split(" ")[0]

	    if ip_count.has_key?(ip_address)
	    	ip_count[ip_address] += 1
	    else
	    	ip_count[ip_address] = 1
	    end
	    
	  end

	 end

	return Hash[ip_count.sort_by{|k, v| v}.reverse]
end

ip_list = read_parse("../apache.log")

puts "RANK\tIP\t\tCOUNT"

ip_list.take(10).each_with_index do |ip, index|
	puts "#{index+1}\t#{ip[0]}\t#{ip[1]}"
end
{% endhighlight %}

* * *

### Python
{% highlight python %}
from operator import itemgetter 

def read_parse(filepath):
	ip_count = {}

	with open(filepath, "r") as file:
		for line in file:
			ip_address = line.split(" ")[0]

			if ip_address in ip_count:
				ip_count[ip_address] += 1
			else:
				ip_count[ip_address] = 1

	return ip_count


ip_list = read_parse("../apache.log")

#The header
print("RANK\tIP\t\tCOUNT")

# Using enumerate() so we can have the iteration's index (rank variable), starting at 1
# Sorting the items of the ip_list by their value in reverse order, but limiting the loop from 0 to 10
# Ps: Python is beautiful =)
for rank, line in enumerate(sorted(ip_list.items(), key=itemgetter(1), reverse=True)[0:10], start = 1):
	print("{}\t{}\t{}\t".format(rank,line[0],line[1]))
{% endhighlight %}

* * *

### Groovy

{% highlight groovy %}
def read_parse(filepath) {
	def thefile = new File(filepath)
	def ipCount = [:]

	thefile.eachLine {
	    def ipAddress = it.split(' ')[0]

	    if(ipCount[ipAddress] == null) {
	    	ipCount[ipAddress] = 1
		} else {
			ipCount[ipAddress]++
		}
	    
	}

	return ipCount.sort { -it.value }
}

def ipList = read_parse("../apache.log")

def rank = 1

print "RANK\tIP\t\tCOUNT\n"

for (ip in ipList) {
    print rank + "\t" + ip.key + "\t" + ip.value + "\n"
    rank++
    if(rank > 10) break
}
{% endhighlight %}

* * *

### C&#35;

(Yes, I've developed this code on my ArchLinux box with MonoDevelop and it ran beautifully)

{% highlight csharp %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.IO.File;

namespace Top10IPs {
	class MainClass {
		public static void Main (string[] args) {
			try {

				Dictionary<string, int> dic = new Dictionary<string, int>();

				string[] lines = ReadAllLines(@"apache.log");
				char[] delimiterChars = { ' ' };

				foreach (string line in lines) {
					string[] lineParts = line.Split(delimiterChars);
					string ipAddress = lineParts[0];

					if(dic.ContainsKey(ipAddress)){
						dic[ipAddress]++;
					} else {
						dic.Add(ipAddress, 1);
					}

				}

				Console.WriteLine("RANK\tIP\t\tCOUNT");

				int i = 1;
				foreach (KeyValuePair<string,int> item in dic.OrderByDescending(key=> key.Value)) {
					Console.WriteLine(i + "\t" + item.Key + "\t" + item.Value);
					i++;
					if(i > 10) break;
				}
			
			} catch (Exception ex) {
				Console.WriteLine (ex.Message);
			}
		}
	}
}

{% endhighlight %}

* * *

The other languages you can check [here](https://github.com/jonathas/top10ips) 

So, this is it for today. Lately I've been working a lot with the [Ionic Framework](http://ionicframework.com), which is a framework built for hybrid mobile development and uses AngularJS and Phonegap/Cordova, and I've also been studying more about Rails and Laravel, so some of my next posts will probably be about these technologies.