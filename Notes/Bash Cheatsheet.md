This is a non exhaustive list of some famous utilities that are available in a Bash environnement. If you think you'll be ask some question about Linux, make sure you know how to use the following tools.

* The goal is be confident with the main bash command 
* Estimated Time: 3 hour

wget: 
* download a file: wget url
* download file and save with different name: wget -O filename url
* download in the background: -b
* some browser can disallow you to download a page by identifying that the agent is not a browser, you can mask the user agent with --user-agent
* download a list of file with wget -i list_url_per_line
* download a full website: wget --mirror
* reject certain files: wget --reject=gif url
* accept only certain files: wget -r -A.pdf url

curl:
* do the same thing as wget, but use a API under the hood, while wget is only a CLI
* curl can't download recursively

wc:
* `-l`: count the number of lines
* `-w`: number of words
* `-m`: number of words

tee: 
* read from standard input and write to standard output (and files): ```echo "" | tee file.txt | wc -l```
* append to a file: `-a`

rsync: 
* synchronize and make a copy whenever there is a new file


pwd:
* print the name of the current folder	

ps:
​* show a snpashot of the current processes


host: 
* convert names to IP addresses and vice versa


nc (netcat): anything that has to do with netcat
* open a connection there nc -l  1234
* connect to this connection nc 127.0.0.1 1234

locate:
* similar as find, but faster. Does not use the file system, but need a database to search in. The database needs to be update with updatedb: `locate filename`

ln:
* hard link create a link to the inode, while a soft link create a link to the file. If the file changes name, hard link keeps working, soft no. if you replace a file with another  (same name), soft link will work, not hard link
* `ln -s /original_path /new_path`

grep:
* grep -E: transformt the expression to match into a normal pattern matching
* grep -F: receive a file of pattern, one per line
* grep -c: count the number of occurences

find:
* Search recursively directory find directory -name filename
* Search directory: -type d

du/df:
* estimate file space usage versus report file system disk space usage

chown:
* change file owner and group (who owns the file)

chmod:
* change who can do what
* `drwxr-xr-x`: directory/owner right/group right/other rights (read=4, write=2, execute=1)

sed: 
* substition: sed 's/old/new/g'
* deletion: sed '/pattern/d'
* combine: sed 's/old/new/g; /pattern/d'	

rev: 
* reverse each line of a file

which:
* locate where is the exec of a command