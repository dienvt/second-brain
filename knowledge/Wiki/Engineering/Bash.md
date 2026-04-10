---
title: "Remove redis key"
date: 2026-01-14
tags:
  - engineering
  - bash
---

  

# Remove redis key

```Bash
#!/bin/sh

# check args count
if [ $# -ne 1 ]; then
        echo "please specify file"
		exit 1
fi

delete(){
    redis-cli --no-auth-warning -h 10.50.49.15 -p 6382 -a 201506e074381f4345391166415e51825a5be43ae3843d022222845c2 DEL $1
}

# Loop all content in file $1
while read line; do
    delete ${line};
done < $1
```

  

How to use

```Bash
base delete.sh file_name.txt
```

  

## Condition statement

```Bash
# condition must be "[ ]" to evaluate conditions.
if condition1
then
	statement1
	statement2
	..........
elif condition2
then
	statement3
	statement4
	........
else
	........
fi
```

  

condition operator

|   |   |   |
|---|---|---|
|operator|produces true if...|number of operands|
|-n|operand non zero length|1|
|-z|operand has zero length|1|
|-d|there exists a directory whose name is _operand_|1|
|-f|there exists a file whose name is _operand_|1|
|-eq|the operands are integers and they are equal|2|
|-neq|the opposite of -eq|2|
|=|the operands are equal (as strings)|2|
|!=|opposite of =|2|
|-lt|_operand1_ is strictly less than _operand2_ (both operands should be integers)|2|
|-gt|_operand1_ is strictly greater than _operand2_ (both operands should be integers)|2|
|-ge|_operand1_ is greater than or equal to _operand2_ (both operands should be integers)|2|
|-le|_operand1_ is less than or equal to _operand2_ (both operands should be integers)|2|

## Loops

```Bash

# for loop in
# {begin..end..increment}
for X in red green blue
do
	echo $X
done


# while conditions wrap in "[ ]"
X=0
while [ $X -le 20 ]
do
	echo $X
	X=$((X+1))
done
```