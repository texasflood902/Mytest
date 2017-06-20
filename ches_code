import subprocess
import os
import sys
import random
import numpy as np

# SET NECESSARY VARIABLES HERE
correctKeyEnable = 0
randomlyGenerateKey = 0
fileName = 'C3540'

dirPath = os.path.dirname(os.path.abspath(__file__))
finalFilePath = dirPath + '/outputs/' + fileName + '_antiSAT.blif'

# ====================> Convert file to bench

fs = open('abc_script_2bench','w')
fs.write('read ' + finalFilePath + '\n')
fs.write('strash \n')
fs.write('write_bench -l toBench.bench \n')
fs.close()

argument1 = ('-F')
argument2 = ('abc_script_2bench')
subprocess.call(['./abc', argument1, argument2])

os.remove('abc_script_2bench')

# ====================> Create Miter

f1 = open('toBench.bench','r')
f2 = open('miter_file.bench','w')

inputList = []
prefix = ['_A','_B']
keyLength = 0

for line in f1:

	words = line.replace('(',' ')
	words = words.replace(')',' ')
	words = words.replace(',',' ')
	words = words.split()

	# Skip Comment
	if '#' in line:
		doNothing = 0

	# Primary inputs (don't make copies)
	elif ('INPUT' in line) and ('keyinput' not in line):
		inputName = str(words[1])
		inputList.append(inputName)
		f2.write(line)

	# Key inputs (make copies)
	elif ('INPUT' in line) and ('keyinput' in line):
		keyInputName = str(words[1])
		keyLength += 1
		for i in range(0,len(prefix)):
			f2.write('INPUT(' + keyInputName + prefix[i] + ')' + '\n')

	
	# Output (rename it as miter output)
	elif 'OUTPUT' in line:
		outputName_int = words[1]
		f2.write('OUTPUT(xorout)\n')

	# Make copies of all gates
	else:
		newWord = ['']*len(words)

		for j in range(len(prefix)):
			for i in range(len(words)):

				# Net name, output
				if (i < 1):
					newWord[i] = words[i] + prefix[j] 

				# Net name, inputs
				elif (i > 2) and (i < len(words) - 1):
					if (words[i] not in inputList):
						newWord[i] = words[i] + prefix[j] + ', '
					# Check if its a primary input					
					else: 
						newWord[i] = words[i] + ', '

				# Net name, last input			
				elif (i == len(words) - 1 ):
					if (words[i] not in inputList):			
						newWord[i] = words[i] + prefix[j] + ')'
					else:
						newWord[i] = words[i] + ')'
				# Gate type
				elif (i == 2):   			
					newWord[i] = words[i] + '('

				elif (i == 1):
					newWord[i] = ' ' + words[i] + ' '

				else:
					print('Something wrong while parsing bench file')
					sys.exit()
				
	
			f2.write(''.join(newWord) + '\n')

f1.close()
f2.close()

# ====================> Append XOR gate to miter copy A and B

f2 = open('miter_file.bench','a')
string = 'xorout = XOR('
for i in range(0,len(prefix)):
	if i != (len(prefix) - 1):
		string = string + outputName_int + prefix[i] + ' , '
	else:
		string = string + outputName_int + prefix[i]

string = string  + ')' + '\n'
f2.write(string)
f2.close()

os.remove('toBench.bench')

# ====================> Declare variables for CNF (primary inputs, gate outputs)

f1 = open('miter_file.bench','r')
dictionary = {}

i = 1 
for line in f1:
	words = line.replace('(',' ')
	words = words.replace(')',' ')
	words = words.replace(',',' ')
	words = words.split()
	if ('INPUT' in words):
		dictionary[words[1]] = i
 		i += 1
		#print(words[1] + ' ' + str(i))

	# Dont assign output variable as its there in the body of netlist
	elif ('OUTPUT' in words):
		outputName = words[1]
		doNothing = 0
	else:
		dictionary[words[0]] = i
 		i += 1
		#print(words[0] + ' ' + str(i))

f1.close()
numVars = i - 1

# ====================> Tseitin Transformation for CNF generation

f1 = open('miter_file.bench','r')
f2 = open('miter_cnf_temp.cnf','w')	

i = 0
for line in f1:
	words = line.replace('(',' ')
	words = words.replace(')',' ')
	words = words.replace(',',' ')
	words = words.split()

	if ('INPUT' in words) or ('OUTPUT' in words):
		doNothing = 0
	else:
		gateType = words[2]
		if (gateType == 'AND'):
			c = str(dictionary[words[0]])
			a = str(dictionary[words[3]])
			b = str(dictionary[words[4]])

			f2.write('-' +  a  + ' '   +   '-' + b + ' '         + c + ' 0' + '\n')
			f2.write(       a  + ' '                         '-' + c + ' 0' + '\n')
			f2.write(                            b + ' '  +  '-' + c + ' 0' + '\n')
			i += 3

			
		elif (gateType == 'XOR'):
			c = str(dictionary[words[0]])
			outputVar = c
			a = str(dictionary[words[3]])
			b = str(dictionary[words[4]])
			f2.write('-' +  a  + ' '   +   '-' + b + ' '  +  '-' + c + ' 0' + '\n')
			f2.write(       a  + ' '   +         b + ' '  +  '-' + c + ' 0' + '\n')
			f2.write(       a  + ' '   +   '-' + b + ' '  +        c + ' 0' + '\n')
			f2.write('-' +  a  + ' '   +         b + ' '  +        c + ' 0' + '\n')
			i += 4


		elif (gateType == 'NOT'):
			c = str(dictionary[words[0]])
			a = str(dictionary[words[3]])
			f2.write('-' +  a  + ' '                      +  '-' + c + ' 0' + '\n')
			f2.write(       a  + ' '                             + c + ' 0' + '\n')
			i += 2
		
		
		else:
			print('Unrecognized gate type, can only handle XOR, AND, NOT')
			sys.exit()

f1.close()
f2.close()
numClauses = i

f1 = open('miter_cnf_temp.cnf','r')	
f2 = open('miter_cnf.cnf','w')	
f2.write('p cnf ' + str(numVars) + ' ' + str(numClauses) + '\n')
for line in f1:
	f2.write(line)

f1.close()
f2.close()

os.remove('miter_cnf_temp.cnf')
#os.remove('miter_file.bench')


####################################################################################################################
######################################################### ALGORITHM START ##########################################
####################################################################################################################

# ====================> Read in CNF

from pycryptosat import Solver
s = Solver()

f1 = open('miter_cnf.cnf','r')
for line in f1:
	sentence = line.split()
	if (sentence[0] == 'p'):
		doNothing = 0
	else:
		clause = [0]*(len(sentence) - 1)
		for i  in range(0,len(clause)):
			clause[i] = int(sentence[i])
		check = s.add_clause(clause)
		if (check != None):
			print('Problem adding clause from CNF file')
			sys.exit()
f1.close()

# ====================> Find key and input variables from dictionary

keyVariables = []
inputVariables = []
for key, value in dictionary.iteritems():
	if ('keyinput' in key):
		keyVariables.append(value)
	elif (key in inputList):
		inputVariables.append(value)
	
keyVariables.sort()
inputVariables.sort()

keyCopy = keyVariables[:]

iteration = 0

lastIndex = dictionary['xorout']
maximumVar = lastIndex - 1

breakFlag = 0

keyVariables = keyCopy[:]


print(' ')
print('Modes: correctKeyEnable: ' + str(correctKeyEnable) + ' randomlyGenerateKey: ' + str(randomlyGenerateKey))
print(' ')

# Find keys that cause different output
outputVar = dictionary['xorout']    	# Force miter output to be 1 (to find DIP)
sat,solution = s.solve([int(outputVar)])
getKeyTF = solution[len(inputVariables) + 1: len(inputVariables) + 1 + len(keyVariables)]

for i in range(0,len(keyVariables)):
	currentKeyVal = getKeyTF[i]

	if (randomlyGenerateKey == 1):
		currentKeyVal = random.choice([True,False])

	if (currentKeyVal == True):
		sign = 1
	else:
		sign = -1

	keyVariables[i] = abs(keyVariables[i]) * sign



# Read in correct key
f1 = open(dirPath + '/outputs/'+fileName + '_key.txt')
i = 0
for line in f1:
	line = line.replace('[','')
	line = line.replace(']','')
	line = line.replace(',','')
	if (i == 1):
		keyString1 = line.split()
	elif (i == 3):
		keyString2 = line.split()	
	i += 1

f1.close()
correctKey = keyString1 + keyString2


# Make one of the keys the correct key

if (correctKeyEnable == 1):
	j = 0
	stringy = ''
	for i in range(0,len(keyVariables)):
		if (i%2 == 0):
			if (correctKey[j] == '0'):
				keyVariables[i] = abs(keyVariables[i]) * (-1)
			else:
				keyVariables[i] = abs(keyVariables[i])
			j += 1

			stringy = stringy + ' ' + str(keyVariables[i])
	
	
# Print out keys currently being used
k1Str = ''
k2Str = ''
for i in range(0,len(keyVariables)):
	if (i%2 == 0):
		k1Str += str(keyVariables[i]) + ' '
	else:
		k2Str += str(keyVariables[i]) + ' '
print('Key 1')
print(k1Str)
print(' ')
print('Key 2')
print(k2Str)



# Find no. of patterns (SAT starts)

outputVar = dictionary['xorout']    	# Force miter output to be 1 (to find DIP)
keyVariables.append(int(outputVar))

print(' ')
print('SAT Results')
print(' ')

solvedIter = 0
while True:

	sat,solution = s.solve(keyVariables)

	if (sat == False):
		print(' ')
		print('UNSAT .. aborting ... No DIPs found')
		#print(keyVariables)
		#print(sat)
		#print(solution)
		break

	else:

		assumption = []  						# Get True/False solution
		assumption = list(solution[1 : len(inputVariables) + 1]) 	# for DIP		

		assumpList = inputVariables 					# Convert True/False to +/-Vars
		for i in range(0,len(inputVariables)):	 			# Get the DIP assumptions
			t_or_f = assumption[i]
			if (str(t_or_f) == 'False'):
				assumpList[i] = (abs(assumpList[i]) * (-1)) * (-1)
			else:
				assumpList[i] = (abs(assumpList[i]) * (-1))

		s.add_clause(assumpList)
		solvedIter += 1
		print(assumpList)
		print('SAT Solver called: ' + str(solvedIter))


	iteration += 1

	if (iteration > 50000):
		exit()

















