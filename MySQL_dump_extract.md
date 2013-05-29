MySQL Dump Extract
==================

I was going through some old source code recently and found a script I used frequently before. The script allows you to simply extract tables from a MySQL dump file; however, if the MySQL dump file is over 5GB, I would not recommend using the script since the source file is read into memory.

I might recode this script in the future using Python to deal with larger files.

```perl
#!/usr/bin/perl -w
#
# Author: Damian Myerscough
# Date: 25/09/2010
# Description: Extract tables from MySQL dump files
#
#
 
use strict;
use Getopt::Std;
use vars qw/ %opt %Tables /;
 
# Enable contents copying
my $Copy = 0;
my $TablePos = 0;
 
# Contains the contents of each table
my @TableContent = ();
 
sub usage()
{
    print <<EOF;
Usage: $0 -d [MYSQL DUMP] -l
Usage: $0 -d [MYSQL DUMP] -a
Usage: $0 -d [MYSQL DUMP] -t [TABLE] -o [TABLE OUTPUT]
 
    -h    : Display this help message
    -d    : MySQL dump file
    -l    : List all tables
    -a    : Extract all tables
    -t    : Select single table to extract
    -o    : Extract single table
EOF
 
    exit(0);
}
 
sub Extract()
{
    # Open and read the whole MySQL dump file
    open(DUMPFILE, "$opt{d}") || die "[*] Unable to $opt{d} file $!\n";
        my @DUMP = <DUMPFILE>;
    close(DUMPFILE);
 
    # Iterate through the log file extracting each table 
    foreach my $line (@DUMP)
    {
        if ( ($line =~ /^-- Table structure for table \`([A-zZ-a _0-9]+)\`/i) )
        {
            $Tables{$1} = ++$TablePos;
 
            # Enable the contents to be written to our array 
            $Copy = 1;
        }
 
        if ($Copy)
        {
            # Copy each line to our array
            $TableContent[$TablePos] .= $line;
        }
 
        if ($line =~ /^UNLOCK TABLES\;/i)
        {
            # Stop copying our data since there is going to be a new table 
            # or we have finished
            $Copy = 0;
        }
    }
}
 
sub tableExtract()
{
    &Extract;
 
    # Check to see if the table exists and the extract the single table
    if ( exists $Tables{$opt{t}} )
    {    
        print "[*] Extracting the $opt{t} table\n";
 
        # Write the table to the users desired destination
        open(DUMPFILE, ">$opt{o}") || die "[*] Unable to create $opt{o} table: $!\n";
            
        foreach my $line (@TableContent[$Tables{$opt{t}}])
        {
            print DUMPFILE "$line\n";
        }
 
        close(DUMPFILE);
    }
}
 
sub extractAll()
{
    &Extract;
 
    # Extract all tables and write them into the present working directory
    for my $Table (keys %Tables)
    {
        print "[*] Extracting $Table\n";
 
        open(DUMPFILE, ">$Table") || die "[*] Unable to create $Table table: $!\n";
            
        foreach my $line (@TableContent[$Tables{$Table}])
        {
            print DUMPFILE "$line\n";
        }
            
        close(DUMPFILE);
    }
}
 
sub listTables()
{
    &Extract;
    print "[*] Listing available tables\n\n";
 
    for my $Table (keys %Tables)
    {
        print "[*] $Table\n";
    }
}
 
getopts( "lhat:d:o:", \%opt ) || usage();
 
usage() if $opt{h};
listTables() if $opt{d} && $opt{l};
extractAll() if $opt{d} && $opt{a};
tableExtract() if $opt{d} && $opt{t} && $opt{o};
```


