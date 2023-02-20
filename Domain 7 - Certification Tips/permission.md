chmod => change modification of file permission

In linux their are 3 type of owner and 3 type of permission

3 type OWNERSHIP

1.user

2.group

3.other

3 type PERMISSION

1.Read

2.Write

3.Excute

so chmod means change the ownership and permission

sudo chmod 400

here 3 digit mention

1st position show ownership of USER === Here 4

2st position show ownership of GROUP === Here 0

3st position show ownership of OTHER === Here 0

And number say

0=>No permission

1=>Excute

2=>Write

3=>Execute + Write

4=>Read

5=>Read + Execute

6=>Read +Write

7=>Read + Write +Execute

So by this

sudo chmod 400

says

for USER it is READ permission

for GROUP it is NO permission

for OTHER it is NO permission

Get more Detail On

File Permissions in Linux/Unix with Example