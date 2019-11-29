# Configuracion de PyModbusTCP
Mas informacion Buscar https://pypi.org/project/pyModbusTCP/
y descargar por medio de https://github.com/Elias-alarcon/PyModbusTCP/blob/master/pyModbusTCP-0.1.8.tar.gz

	#!/usr/bin/env python3.7
	from pyModbusTCP.client import ModbusClient
	from time import sleep
	from datetime import datetime
	import http.client, urllib
	import os,time,random
	from progress.bar import IncrementalBar   
	c = ModbusClient(host='192.10.2.101', auto_open=True)
	c.open()

	def pres_torre ():
		P_T = c.read_input_registers(4,1) #conversion de lista a enteros
		c_p_t=" ".join(str(e) for e in P_T)
		Press_torre = int(c_p_t)
		return Press_torre
	def pres_chiller():
		P_ch = c.read_holding_registers(5,1)
		c_p_ch=" ".join(str(e) for e in P_ch)
		Press_chiller = int(c_p_ch)
		return Press_chiller
	def pres_comp():
		p_cp= c.read_holding_registers(7,1)
		c_p_cp=" ".join(str(e) for e in p_cp)
		Press_compresor = int(c_p_cp)
		return Press_compresor
	def temp_chiller():
		T_ch= c.read_holding_registers(6,1)
		c_t_ch=" ".join(str(e) for e in T_ch)
		Temp_chiller = int(c_t_ch)
		return Temp_chiller
	def temp_torre():
		t_t= c.read_holding_registers(8,1)
		c_t_t=" ".join(str(e) for e in t_t)
		Temp_torre = int(c_t_t)
		return Temp_torre
			#valores de radio
	def z_axis():
		Z_a=c.read_holding_registers(17,1)
		c_Z_a=" ".join(str(e) for e in Z_a)
		Z_axis= (int(c_Z_a)/1000)
		return Z_axis
	def x_axis():
		X_a=c.read_holding_registers(21,1)
		c_X_a=" ".join(str(e) for e in X_a)
		X_axis= (int(c_X_a)/1000)
		return X_axis
	def temp_motor():
		t_mot=c.read_holding_registers(18,1)
		c_t_mot=" ".join(str(e) for e in t_mot)
		Temp_motor= (int(c_t_mot)/20)
		return Temp_motor
	def alarma():
		v_ala= c.read_holding_registers(10,1)
		c_v_ala=" ".join(str(e) for e in v_ala)
		Valor_ala= int(c_v_ala)
		return Valor_ala
	def loading():
		 bar = IncrementalBar('___Recolección de datos___', max=100)# barra de progreso de envio de datos 
		 for i in range(100):
			sleep(.6)
			bar.next()
         bar.finish()
	p_t=pres_torre()
	p_c=pres_chiller()
	p_cp=pres_comp()
	t_c=temp_chiller()
	t_t=temp_torre()
	z_a=z_axis()
	x_a=x_axis()
	t_m=temp_motor()
	v_a=alarma()

	def envio_nube(): #modulo de envio


		if c.is_open:
				print("preparando envio de datos")
				loading()
				params = urllib.parse.urlencode({'field1': p_t,'field2':p_c,'field3': p_cp,
												 'field4': t_c,'field5': t_t,'field6': str(z_a),
												 'field7': str(x_a),'field8': str(t_m),
												 'key':'23GN8QPWNXNTXWMV'})
				headers = {"Content-typZZe": "application/x-www-form-urlencoded","Accept": "text/plain"}
				conn = http.client.HTTPConnection("api.thingspeak.com:80")

				try:
				 conn.request("POST", "/update", params, headers)
				 response = conn.getresponse()

				 print(response.status, response.reason)
				 data = response.read()
				 conn.close()
				except:
				 print("connection failed")
				 lecturas_horus()


				print('Envio de datos exitoso')
				sleep(3)

	def lecturas_horus():

		print(' ___ LECTURA ___ ' + str(datetime.now()))

		if not c.open():
				print("unable to connect to "+SERVER_HOST+":"+str(SERVER_PORT))
		if c.is_open():

			print('Presion Torre= '  + str (p_t)+ ' PSI' )
			print('Presion Chiller= '  + str (p_c)+ ' PSI' )
			print('Presion Compresor= '  + str (p_cp)+ ' PSI' )
			print('Temperatura Torre= '+ str (t_t) +' °F')
			print('Temperatura Chiller= '+ str (t_c) +' °F')
			print('Temperatura Motor= '+ str (t_m) +' °F')
			print('Vibracion  Z_axis = '+ str (z_a) +' in/s')
			print('Vibracion  X_axis = '+ str (x_a) +' in/s')

			if v_a ==0 :
				print ('Alarma Desactivada')

			else:
				print('Alarma Activada')

			print('Sin envio de datos a la nube')
			sleep(3)
