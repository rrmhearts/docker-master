# doqueru: snail:
This has been translated from https://github.com/joseims/doqueru-kun/blob/master/README.md along with his repo. The goal is to have all of my Docker information in one place. 

# Proposal  
Based on [this tutorial] (http://cesarvr.github.io/post/2018-05-22-create-containers/?fbclid=IwAR115qJ_sKet0uQM3fJ6u1ALe9JHpEOldX4lE-HWVF_Fm-P0ctf6P9DcHJM) joseims created his own application programs for containerization. The features that he implemented are:

## Features
: snail:: File system isolation  
: snail:: User namespace isolation  
: snail:: Isolation of variables  
: snail:: Isolation of the process tree and network interfaces  
: snail:: Memory usage control  
: snail:: In% (hard or soft)  
: snail:: CPU usage control  
: snail:: Use pivot_root  
: snail:: Allow disk control  
: snail:: Limit / block systemcalls

## Usage

> dokeru [* OPTION *] ...
>
> - shares * CPU_SHARES *  
> & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp;
> define value of CPU_SHARES  
> - period * CPU_PERIOD *  
> & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp;
> sets the value of CPU_PERIOD  
> - percent * CPU_PERCENT *  
> & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp;
> sets CPU_PERCENT value to a number from 1 to 100  
> - cpus * CPU_CPUS *  
> & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp;
> define value of CPU_CPUS  
> - memlimit * MEM_MAX *  
> & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp; & nbsp;
> sets the value of MEM_MAX  

## Notes
* When `unshare -m` is followed by an` exit` 
* https://yarchive.net/comp/linux/pivot_root.html
* https://lists.gnu.org/archive/html/guix-commits/2015-06/msg00047.html
* https://news.ycombinator.com/item?id=11558364