#!/bin/python3.8
import sys, os.path
from bs4 import BeautifulSoup
from requests import Request, Session
from json import loads as loadJsonString

class Reader():
	"""Todo a lógica da sintaxe da linguagem ficara aqui e definição das coisas"""

	def __init__(self, file):
		self.__reserved_words = [
			'BaseURL'
		]

		self.__file = file
		self._url = None
		self.readFile()
	
		self.setCommands()
		self.setURL()

		print(self.getURL())
		
		pass

	def readFile(self):
		with open(self.__file, 'r') as scrapfile:
			self.__lines_file = [line.replace('\n', '') for line in scrapfile.readlines()]
		pass

	def setURL(self):
		for command in self.getCommands():
			if 'BaseURL' in command:
				
				self.__url = self.getValueVar(command)
				break
		pass

	def getURL(self):
		return self.__url

	def getValueVar(self, command):
		# Vai precisar refatorar
		def_value = command.split('<-')[1]

		if def_value.find('"') == -1:
			start = def_value.find('`')+1
			end = def_value.rfind('`')

			value_var = def_value[start:end]
		else:
			start = def_value.find('"')+1
			end = def_value.rfind('"')

			value_var = def_value[start:end]

		return value_var

	def setCommands(self):
		self.__commands = list()
		number_of_lines = len(self.__lines_file)

		start = 0
		command = str()

		for line_n in range(number_of_lines):
			if "&&" not in self.__lines_file[line_n].replace(' ', '')[0:3]:
				command += self.__lines_file[line_n]

			if self.__lines_file[line_n].endswith(';') and "&&" not in self.__lines_file[line_n]:
				self.__commands.append(command)
				command = str()
				start = line_n+1

		# print(self.getCommands())
		pass

	def filterJson(self, command_to_filter):
		data = command_to_filter

		json_start = command_to_filter.find('{')
		json_end = command_to_filter.rfind('}')+1
		json_str = command_to_filter[json_start:json_end]

		return loadJsonString(json_str)

	def getCommands(self):
		return self.__commands

class Code(Reader):
	def __init__(self, file):
		super().__init__(file)

		self.__session = Session()
		self.setDefaultHeaders()
		# Baseado no numero da linha do começo do comando, ele cria um fluxo de trabalho de forma executar primeiro os primeiros comandos
		# Pra executar de maneira sequencial, use sorted(dict) e use a lista no for loop
		self.__work_flow = dict()

		self.setWorkFlow()
		
		self.executar()
		pass

	def setWorkFlow(self):
		commands = self.getCommands()
		for (command, i) in zip(commands, range(len(commands))):
			if 'GET' in command or 'POST' in command:
				command_code = self.createRequestCommand(command)
			
				self.__work_flow[i] = command_code
			else:
				# create_command()
				pass
		pass

	def getRequestProps(self, request_props):
		path_start = request_props.find('"')+1
		path_end = request_props.rfind('"')
		
		method = request_props[0:request_props.find(' ')]
		path = request_props[path_start:path_end]

		if "http" in path[:6]:
			url = path
		else:
			url = self.getURL() + path

		return method, url

	def createRequestCommand(self, request_code):
		request_props = request_code.split(',')[0]
		request_header = request_code.split(',')[1:]

		method, url = self.getRequestProps(request_props)
		headers = self.getRequestHeadersFromCommand(request_header)
		# Aqui ele ta fazendo muita coisa
		#Get Request Data to Send 
		try:
			data = headers.pop('data')
		except KeyError:
			data = {}

		try:
			query = headers.pop('query')
		except KeyError:
			query = {}

		return (Request(method, url, headers=headers, params=query, data=data).prepare(), 'request')

	def setDefaultHeaders(self):
		commands = self.getCommands()
		for command in commands:
			if "Headers" in command:
				props = command.split(':')[1].split(',')
				
				headers = self.getRequestHeadersFromCommand(props)

				self.__session.headers.update(headers)

				break
		pass

	def getRequestHeadersFromCommand(self, command_line_list):
		header_dict = dict()
		for line in command_line_list:
			if '->' in line:
				props = line.split('->')

				name = props[0].replace(' ', '').lower()
				# Talvez precisa de refatoramento
				# Get value from Command?
				if "(JSON)" in props[1]:
					value = self.filterJson(props[1])
				else:
					value_start = props[1].find('"')+1
					value_end = props[1].rfind('"')
					value = props[1][value_start:value_end]

				header_dict[name] = value

		return header_dict

	def executar(self):
		for key_item in self.__work_flow.items():
				item = key_item[1]
				if item[1] == 'request':
					print(self.__session.send(item[0]).url)
				# print(item[0].headers)
		pass


def test_parse():
	page_test = """<html>
	<a class='teste' id='4321'>Teste 1</a>
	<a class='teste' id='3214'>Teste 2</a>
	<a class='teste' id='1234'>Teste 3</a>
</html>\n"""
				
	print("HTML -> \n" + page_test)
	print("Class of first tag <a> with class teste:\n" + str(getDataByDict(page_test, tag='a', filter={'class':"teste"}, value='class')) + "\n")
	print("Id of first tag <a> with class teste\n" + str(getDataByDict(page_test, filter={'class':"teste"}, value='id')) + "\n")
	print("Text of tag <a> with id 1234 \n" + str(getDataByDict(page_test, filter={'id':"1234"}, value='text')) + "\n")
	print("Show all tags only\n" + str(getDataByDict(page_test, tag='a', value='text', all=True)) + "\n")
	print("Show all tags\n" + str(getDataByDict(page_test, tag='a', all=True)) + "\n")

def getDataByDict(html, **filterDict):
	# Use get para evitar erros de não atribuição 
	#filterDict['filter']
	page_test = BeautifulSoup(html,"html.parser")
	
	nameTag = filterDict.get('tag')
	tagFilter = filterDict.get('filter')

	if nameTag == None:
		if filterDict.get("all"):
			data = page_test.find_all(attrs=tagFilter)
			return data
		else:
			data = page_test.find(attrs=tagFilter)
	else:
		if filterDict.get("all"):
			data = page_test.find_all(nameTag, attrs=tagFilter)
			return data
		else:
			data = page_test.find(nameTag, attrs=tagFilter)

	if filterDict.get('value') == None:
		return data
	else:
		if filterDict['value'] == 'text':
			dataValue = data.text
		else:
			dataValue = data.get(filterDict['value'])

	return dataValue

if __name__ == '__main__':
	if '--test_parse' in sys.argv:
		test_parse()
	elif '--test'  in sys.argv:
		Code('teste')
		
	else:
		file = sys.argv[1]
		if os.path.exists(file):
			Code(file)
		
