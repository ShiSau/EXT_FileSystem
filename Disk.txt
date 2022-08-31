import Disk
import os


DISK_FILE	= "disk.dmg"
BLOCK_SIZE	= 512
curdir = ["/"]
curblock = [66]
my_dict = {}
blockindex = 0

# two types of inodes
TYPE_DIR = 0x1111
TYPE_FILE = 0x2222

# indicates an unused directory listing
DIR_FREE = 0xffff


class my_dictionary(dict):

    def __init__(self):
        self = dict()

    def add(self, key, value):
        self[key] = value


# class reverseBytes:

#     def __init__(self):
#         self.revlist = []
# 	def reverse(self, mylist, revlist):
# 		return mylist


class Disk:

	# Disk constructor
	#
	# @param diskfile	Name of the disk file to be opened. Should
	#					be a multiple of `blocksize` in size
	# @param blocksize	Block size to use
	#
	# @return none
	def __init__(self, diskfile, blocksize):

		try:
			self.diskfile = open(diskfile, "r+b")
		except IOError:
			raise Exception("Failed to open disk file")

		self.blocksize = blocksize
		self.blockreads = 0
		self.blockwrites = 0

	# Disk destructor
	#
	# @return none
	def __del__(self):

		self.diskfile.close()

	# Read a block from the disk
	#
	# @param n	Block number to read
	#
	# @return	`blocksize` number of bytes, from the block
	def readBlock(self, n):

		self.diskfile.seek(self.blocksize * n)
		data = self.diskfile.read(self.blocksize)

		self.blockreads += 1

		return data


	# Write a block to the disk
	#
	# @param n		Block number to write to
	# @param data	Data to be written to that block
	#
	# @return none
	def writeBlock(self, n, data):

		self.diskfile.seek(self.blocksize * n)
		self.diskfile.write(data)

		self.blockwrites += 1
	

	# Print statistics showing number of block read/writes
	#
	# @return none
	def printStats(self):

		print("")
		print("===== Disk usage statistics =====")
		print(" Total block reads:  {}".format(self.blockreads))
		print(" Total block writes: {}".format(self.blockwrites))
		print("=================================")
		print("")

# Reverse the given bytes
#
# 
# @param size	Data to be written to that block
#
# @return reversedbytes
def reverseBytes(size):
	updatedSize = []
	total = '0'
	updatedSize = size[::-1]

	for ele in updatedSize:
		total += (ele)
	total = total.replace('0x', '')

	return (int(total,16))


# Write a block to the disk
#
# @param n		Block number to write to
# @param data	Data to be written to that block
#
# @return none
def dataBlock(block):
	metadata = my_dictionary() #key is index 0,1,2...
	imgdata = my_dictionary()  #key is name, value-all the data
	imgdata_namelocation = my_dictionary() # key is name, value-only locations
	locations = []

	# Instantiate a new Disk, backed by the user-supplied file
	d = Disk(DISK_FILE, BLOCK_SIZE)
	metadata = inodesList(d.readBlock(2))

	for x in range(0, 512, 32):
		inode = block[x]		#2 bytes
		if(inode == 255):
			continue
		name = [] # 32 bytes
		name = block[x:x+24].decode('unicode_escape').strip()
		if inode in metadata: 
			name = name.replace("\x00", "").replace("\x07", "").replace("\x02", "").replace("\x01", "")
			name = name.replace("\x03", "").replace("\x05", "").replace("\x04", "").replace("\x06", "").replace("\x08", "")
			imgdata.key = name
			imgdata.value = metadata[inode]
			imgdata.add(imgdata.key, imgdata.value)
			imgdata[name].append(inode)

			imgdata_namelocation.key = name
			locations = [imgdata[name][3], imgdata[name][4], imgdata[name][5], imgdata[name][6]]
			imgdata_namelocation.value = locations
			imgdata_namelocation.add(imgdata_namelocation.key, imgdata_namelocation.value)
			print(metadata[inode][0], end='   ')
			print(metadata[inode][1], end='   ')
			if(int(metadata[inode][2]) !=0 ):
				print(metadata[inode][2], end='  ')
			else:
				print(end='       ')
			print(name)
		

  
		print("\n")
	return imgdata_namelocation, imgdata

# Prints data from indirect block
#
# @param n		Block number to write to
#
# @return none
def indirectData(block):
	for x in range(0, 512):
		while block[x]!=0:
			print(d.readBlock(66+ (block[x])).decode('unicode_escape').strip())
			x= x+1

		


# Inodes data
#
# @param n		Block number to read
# @param my_dict  store files and directories with their block numbers
#             
# @return my_dict
def inodesList(block):
	my_dict = my_dictionary()
	k = 0
	type = ''

	for x in range(0, 512, 16):
		name = ''
		if(block[x] == 0x11):
			type = 'DIR' 
			# print("DIR", end='  ')
		elif block[x] == 0x22:
			type = 'File' 
			# print("File", end='  ')
		else:
			continue

		## ---> links
		linkamt = 0
		for x in range(x+2, x+4):
			linkamt += (block[x])
		# print((linkamt), "link", end=' ')
		## ---> end of links


		## ---> Calculate size 
		size = []
		sizeTotal = 0
		for x in range(x+1, x+5):
			size.append(hex(block[x]))
		sizeTotal = reverseBytes(size)
		# print(sizeTotal, "bytes", end=' ')
		## ---> End of size calculation


		## ---> directs
		nodes = []
		direct1 = 0
		for x in range(x+1, x+3):
			if(block[x] != 00):
				nodes.append(hex(block[x]))
		direct1 = reverseBytes(nodes)
		# print((direct1), " direct1", end='   ')
		node2 =  []
		direct2 = 0
		for x in range(x+1, x+3):
			if(block[x] != 00):
				node2.append(hex(block[x]))
		direct2 = reverseBytes(node2)
		# print((direct2), " direct2", end='   ')
		node3 = []
		direct3 = 0
		for x in range(x+1, x+3):
			if(block[x] != 00):
				node3.append(hex(block[x]))
		direct3 = reverseBytes(node3)
		# print((direct3), " direct3", end='   ')
		## ---> end of directs

		## ---> Indirects
		indirects = []
		indirectsSum = 0
		for x in range(x+1, x+3):
			if(block[x] != 00):
				indirects.append(hex(block[x]))
		indirectsSum = reverseBytes(indirects)
		# print((indirectsSum), " Indirects", end='   ')
		## ---> end of Indirects

		my_dict.key = k
		my_dict.value = [type, linkamt, sizeTotal,direct1, direct2, direct3, indirectsSum]
		my_dict.add(my_dict.key, my_dict.value)
		k =k+1

	return my_dict




if __name__ == "__main__":

	# Instantiate a new Disk, backed by the user-supplied file
	d = Disk(DISK_FILE, BLOCK_SIZE)
	my_dict = my_dictionary()
	my_dict2 = my_dictionary()
	x=0

	print("Browsing:  {}".format(DISK_FILE))
	print("Disk label: %s" % d.readBlock(0).decode('unicode_escape').strip())
	print("\n")

	while True:
		print("[DISK_FILE]:", *curdir, end='>')
		commands = input().split()
		if commands[0] =="dir":
				my_dict, my_dict2 = dataBlock(d.readBlock(curblock[blockindex]))
		if commands[0] =="pwd":
				##update curdir values 
				print('home', *curdir)
		if commands[0] =="cd":
			## .. goes to parent  directory 
			if commands[1] == '..':
				if len(curdir) != 0:
					del curdir[-1]
					del curblock[-1]
					blockindex = blockindex-1
				else:
						continue
			elif str(commands[1]) in my_dict:
				curdir.append(str(commands[1]+"/"))
				curblock.append(66+ int(my_dict[commands[1]][0]))
				blockindex = blockindex+1
			else: 
				print("Not a directory:", str(commands[1]))
		if commands[0] =="stat":
			## .. goes to parent  directory 
			if str(commands[1]) in my_dict2:
					print("Name  :", str(commands[1]), "\n")
					print("Inode :",int(my_dict2[commands[1]][7]), "\n")
					print("Type  :", str(my_dict2[commands[1]][0]), "\n")
					print("Links :", int(my_dict2[commands[1]][1]), "\n")
					print("Size  :",int(my_dict2[commands[1]][2]), "\n")
					print("Directs :", int(my_dict2[commands[1]][3]),int(my_dict2[commands[1]][4]),int(my_dict2[commands[1]][5]), "\n")
					print("Indirects :", int(my_dict2[commands[1]][6]), "\n")
					
			else: 
				print("No stat:", commands[1])                 
		if commands[0] =="read":
			if str(commands[1]) in my_dict:
				print(d.readBlock(66+ int(my_dict[str(commands[1])][0])).decode('unicode_escape'))
				while int(my_dict[str(commands[1])][x]) != 0 and x<3:
						print(d.readBlock(66+ int(my_dict[str(commands[1])][x])).decode('unicode_escape'))
						x=x+1
				if int(my_dict[str(commands[1])][3]) != 0:
					indirectData(d.readBlock(66+ int(my_dict[str(commands[1])][3])))
			else:
					print("Wrong filename:", commands[1])
		if commands[0] =="x":
				break
		if commands[0] == "help":
			print("Commands:\n"
					"dir         : Print the contents of the current directory\n"
					"cd <dir>    : Change to directory <dir> \n"
					"read <file> : Print the contents of <file> \n"
					"pwd         : print working directory\n"
					"stat <file> : Print the inode information for this <file> \n")

		print("\n\n")

		d.printStats()


