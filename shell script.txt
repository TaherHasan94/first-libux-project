
msg="-------------------------------------------------- 
select one from the list
1. Show or print student records (all semesters).
2. Show or print student records for a specific semester.
3. Show or print the overall average.
4. Show or print the average for every semester.
5. Show or print the total number of passed hours.
6. Show or print the percentage of total passed hours in relation to total F and FA
hours.
7. Show or print the total number of hours taken for every semester.
8. Show or print the total number of courses taken.
9. Show or print the total number of labs taken.
10. Insert the new semester record.
11. Change in course grade.
0. Exit
--------------------------------------------------
"
#--------------------->>> here we will get the name of the file 
while $true
do

echo -n "enter file student name: "
read fileName

#--------------------->>> check if the file exist 
if [ ${#fileName} -eq 0 ] || [ ! -f $fileName ]
then  
	echo "($fileName) is not existed. please try again . . ."
	echo
else
	break

fi

done



while true
do
echo "$msg"
read choice 
#------------------------------->>> here it will read the choices that user want
if [ $choice -eq 0 ]
then 
	break

elif [ $choice -eq 1 ]
then

cat $fileName

elif [ $choice -eq 2 ]
then
echo -n "Enter the semester you want to show : "
read semester

cat $fileName | grep "$semester"


elif [ $choice -eq 3 ]
then
#------------------------------->> in this case we want to get the average of all semesters
sum=0
hours=0
for courseUniq in $(cat $fileName | sed 's/; \|, \| /\n/g' | sed '/-\|^$\|F\|I\|^[1-9]/d' | sort | uniq | sed 's/ //g')
do
#------------------------------->> this for loop will get uniq courses with handling courses with I grades
        max=0
        for courseWithGrade in $(cat $fileName | sed 's/; \|, /\n/g' | sed '/-\|^$\|I/d' | sed 's/FA/50/g' | sed 's/F/55/g' | sed 's/ //g' |grep $courseUniq)
        do
		#--------------->> this for loop will get the all repeated courses and comparing it to get MAX grade and grade HOUR
		
                gradePosition=$((${#courseWithGrade}-2))
                grade=${courseWithGrade:$gradePosition:2}

                if [ $grade -gt $max ]
                then
                        max=$grade

                fi
		hour=${courseWithGrade:5:1}
		echo "the course with grade =($courseWithGrade), course grade =($grade), course hour =($hour) and max =($max)" 
        done
#------>>in max we will get the real weight of the grade
max=$((max * hour)) 
hours=$((hours + hour))
sum=$((sum + max))

echo "total weight of this course =($max), total hours till now =($hours) and total sum till now =($sum) "
echo "-------------------------------------------------"
done
sum=$((sum / hours))

echo "===================>> The total average of all courses = ($sum)% <<==================="



elif [ $choice -eq 4 ]
then

for semester in $(cat $fileName | sed '/^$/d' | cut -c1-11)
do
sum=0
hours=0
for courseWithGrade in $(cat $fileName | grep $semester| sed 's/; \|, /\n/g' | sed '/-\|^$\|I/d' | sed 's/FA/50/g' | sed 's/F/55/g' | sed 's/ //g')
do
	gradePosition=$((${#courseWithGrade}-2))
	grade=${courseWithGrade:$gradePosition:2}
	hour=${courseWithGrade:5:1}
	weight=$((grade * hour))
	sum=$((sum + weight))
	hours=$((hours + hour))
	echo "course With Grade Form = ($courseWithGrade)  hour = ($hour)  grade = ($grade) weight = ($weight)   sum =($sum)  hours =($hours)"
	echo
done
sum=$((sum / hours))

echo "===================>> the average of cources at ($semester) is ($sum)% <<==================="
echo
done


elif [ $choice -eq 5 ]
then

hours=0
for course in $(cat $fileName | sed 's/; \|, /\n/g' | sed '/^$\|-\|I\|F/d' | tr ' ' '\12' | sed '/^[1-9]\|^$/d' | sort | uniq)

do
	hour=${course:5:1}
	hours=$((hours + hour))
	echo "$course ==> has $hour hours , and total passed hour till now =($hours)"
done
echo
echo "===================>> Total passed hours = ($hours) <<==================="
echo

elif [ $choice -eq 6 ]
then 
hours=0 #---->> passed hours
for course in $(cat $fileName | sed 's/; \|, /\n/g' | sed '/^$\|-\|I\|F/d' | tr ' ' '\12' | sed '/^[1-9]\|^$/d' | sort | uniq)

do
		#-----------> to get the course hour
        hour=${course:5:1}   
        hours=$((hours + hour))
        echo "$course ==> has ($hour) hours , and total passed hour till now =($hours)"
done
echo "total passed hours =($hours)"
echo

totalFail=0 #----->> total fail Hours
for fail in $(cat $fileName | sed 's/; \|, /\n/g' | sed 's/ //g' | sed '/[0-9]$\|^$\|I$/d')
do
		#-----------> to get the course hour
        failHour=${fail:5:1} 
        totalFail=$((totalFail + failHour))
        echo "course ${fail:0:${#fail}-2} ==> has ($failHour) hours and total fail hours till this course =($totalFail)"
done
echo "total fail hours =($totalFail)"
echo
if [ $totalFail -gt 0 ]
then
echo "===================>> The percentage = $(($((hours / totalFail)) * 100 ))% <<==================="
else
echo "total Fail is equal to zero division by 0 "
fi

elif [ $choice -eq 7 ]
then
hours=0
for semester in $(cat $fileName | sed '/^$/d' | cut -c1-11)
do
	hours=0
	for course in $(cat $fileName |grep $semester|sed 's/; \|, \| /\n/g' | sed '/^[1-9]/d')
	do
		hour=${course:5:1}
		echo "$course has ($hour) hours-"
		hours=$((hours + hour))
	done
	echo
	echo "===================>> in $semester the number of total hours taken =($hours) <<==================="
	echo
done

elif [ $choice -eq 8 ]
then

echo "the total number of cources is : $(cat $fileName | sed 's/; \|, /\n/g' | sed '/I$\|-/d'  | tr ' ' '\12' | sed '/^[1-9]\|^F\|^$/d' | sort | uniq | wc -l) cources"


elif [ $choice -eq 9 ]
then

num=0
for course in $(cat $fileName | sed 's/; \|, /\n/g' | sed '/-\|I$/d' | tr ' ' '\12' | sed '/^[1-9]\|^F\|^$/d' | sort | uniq)
do
	if [ ${course:5:1} -eq 1 ]
	then
		num=$((num + 1))
	fi
done
echo "===================>> The total lab courses is : ($num) <<==================="


elif [ $choice -eq 10 ]
then
echo -ne "put here the Form of semester for example (2021-2022/3) : "
read semester
echo

hours=0
index=0
#----> this loop for course and its grade
while true 
do

#----> this loop for the course code
while true 
do

echo -ne "put here the name of the course with its code like this (ENCS3310) : "
read course

if [ ${course:0:4} == "ENCS" ] || [ ${course:0:4} == "ENEE" ]
then 
	if [ ${#course} -eq 8 ] && [ ${course:4:4} -le 5999 ] && [ ${course:4:4} -ge 2000 ]
	then	
		
		hours=$((hours + ${course:5:1}))
		name[$index]="$course"
		break
	else
		echo "course grade must be between 2000 - 5999 try again . . . "
		echo
	fi
else 
echo "you can add only either 'ENEE' or 'ENCS' try again . . ."
echo
fi

done


#-------------------------------------------------------------------------------- the course grade ------------------------------------------------- 

while true
do

echo -ne "the mark : "
read mark
if [ ${#mark} == 1 ] || [ ${#mark} == 2 ]
then	
	
	if [ $mark == F ] || [ $mark == FA ] || [ $mark == I ]
	then	
		marks[$index]=$mark
		echo "${name[$index]} with grade = ${marks[$index]} is added"
		break
	
	elif [ $mark -le 99 ] && [ $mark -ge 60 ]
	then
		marks[$index]=$mark
		echo "${name[$index]} with grade = ${marks[$index]} is added"
		break
	else 
		echo "course grade must be F,FA,I or between 60 and 99 , try again . . . "
	fi
else 
		echo "you have entered unvalid grade try again . . . "	
fi 
done


index=$((index + 1))
echo "you have enterd $index courses with $hours hours"
echo

#--------------------------------------------------------------------------------

echo "if you finished please press X or x to Save or (N) to continue"
read finish

if [ $finish == X ] || [ $finish == x ]
then
	if [ $hours -ge 12 ]
	then
		break
	else
		echo "You still havn't reach 12 hours yet you must enter more"
	fi
fi
done


echo "it will be saved as this :"
echo -ne "$semester; ${name[$i]} ${marks[$i]}"
i=1

while [ $i -ne $index ]
do
echo -ne ", ${name[$i]} ${marks[$i]}" 

i=$((i+1))
done
echo

echo "are you sure you want to add it to the file and save it ? press Y to accept "
read sure


i=0


if [ $sure == Y ] || [ $sure == y ]
then 
echo -ne "$semester; ${name[$i]} ${marks[$i]}" >> $fileName
i=1

while [ $i -ne $index ]
do
	echo -ne ", ${name[$i]} ${marks[$i]}" >> $fileName
	i=$((i+1))
done
echo "" >> $fileName
else 
	echo "not saved"
fi

elif [ $choice -eq 11 ]
then
echo -n "Give me a name of semester you want to change the course in : "
read semester
echo -n "Give me the code of the course : "
read course 
echo -n "Give here the new grade : "
read grade
echo -n "" >  copyFile
while read -r line
do
	#------->>>> we will check the file line by line to get the semester
	if [ `echo $line | grep -c "$semester"` -gt 0 ]  
	then
		echo "we have found the semester"
											#-------------->>>> in this line we can edit the grade of the course and replace it by the new one
		echo -n $line | sed  "s/$course [0-9][0-9]/$course $grade/" | sed "s/$course FA/$course $grade/" | sed "s/$course [IF],/$course $grade,/" >> copyFile
		echo "" >> copyFile
	else 
		echo -n $line >> copyFile
		echo "" >> copyFile
	fi
	
	
done < $fileName
cat copyFile
echo ""
echo -n "the new changes as showed below are you sure to save it at $fileName press (S) to save it : "
read con
if [ $con == S ] || [ $con == s  ]
then
	cp copyFile $fileName
	rm copyFile
else
	echo "didn't save . . . "
fi


fi

done
