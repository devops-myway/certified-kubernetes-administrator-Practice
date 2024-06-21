

##### Rules for Naming variable name
Variable name must begin with alphanumeric character or underscore character (_)

``````sh
HOME
SYSTEM_VERSION
vech
no
# Do not put spaces on either side of the equal sign when assigning value to variable
no=10

# Any of the following variable declaration will result into an error such as command not found:
no  =10
no=  10
no  =  10

# Variables names are case-sensitive, just like filenames.
no=10
No=11
NO=20
nO=2

``````

##### All are different variable names, to display value 20 you've to use $NO variable:

``````sh
echo "$no" # print 10 but not 20
echo "$No" # print 11 but not 20
echo "$nO" # print 2 but not 20
echo "$NO" # print 20

``````

##### Create Your First Bash Script

``````sh


``````
