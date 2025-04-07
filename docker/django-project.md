# Команды Docker

- запустить контейнер с автоматическим перезапуском
sudo docker run -d --name myimage_container -p 8000:8000 --restart unless-stopped myimage 

- собрать образ 
	docker build -t myimage .
   

- сделать архив образа для загрузки на сервер
	docker save -o myimage.tar myimage

	docker load -i /path/to/destination/myimage.tar
   
============================
sudo docker stop mydjangoapp


sudo docker rmi $(sudo docker images -q) удаление всех образов
 
============================================
docker build -t tsn_django .

docker-compose up

редактировал manage.py открыл notepad -> правка -> формат конца строк -> преобразовать UNIX


зайти в контейнер и изменить файлы
sudo docker exec -it app_django_cont /bin/bash
sudo docker restart tsn_django_cont
docker save -o app_django.tar app_django

docker load -i app_django.tar 
============================================
docker build -t tsn_django .
docker save -o tsn_django.tar tsn_django
docker load -i tsn_django.tar
sudo docker run -d --name tsn_django_cont -p 8000:8001 -v /home/sar-bc/sites/data_tsnzv/static:/app/static -v /home/sar-bc/sites/data_tsnzv/media:/app/media -v /home/sar-bc/sites/data_tsnzv/.env:/app/tsn/.env --restart unless-stopped tsn_django
sudo docker run -d --name tsn_django_cont -p 8000:8001 --user root -v /home/sar-bc/sites/data_tsnzv/static:/app/static -v /home/sar-bc/sites/data_tsnzv/media:/app/media -v /home/sar-bc/sites/data_tsnzv/.env:/app/tsn/.env --restart unless-stopped tsn_django


docker run -d --name tsn_django_cont -p 8000:8001  tsn_django

удалить sudo docker rm tsn_django_cont
зайти в контейнер и изменить файлы
sudo docker exec -it tsn_django_cont /bin/bash
sudo docker stop tsn_django_cont
sudo docker restart tsn_django_cont

sudo cp  /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/tsnzv.sar-bc.ru.conf
sudo nano /etc/apache2/sites-available/tsnzv.sar-bc.ru.conf

<VirtualHost *:80>
    ServerName tsnzv.sar-bc.ru

     Alias /static/ /home/sar-bc/sites/data_tsnzv/static/
   <Directory /home/sar-bc/sites/data_tsnzv/static/>
       Options Indexes FollowSymLinks
       AllowOverride None
       Require all granted
   </Directory>

	 Alias /media/ /home/sar-bc/sites/data_tsnzv/media/
	   <Directory /home/sar-bc/sites/data_tsnzv/media/>
		   Require all granted
	   </Directory>
	   
	   ProxyPass / http://localhost:8000/
    ProxyPassReverse / http://localhost:8000/
</VirtualHost>


sudo a2ensite tsnzv.sar-bc.ru.conf sudo a2dissite tsnzv.sar-bc.ru.conf
sudo service apache2 restart

   sudo chown -R www-data:www-data /home/sar-bc/sites/data_tsnzv/static
   sudo chmod -R 755 /home/sar-bc/sites/data_tsnzv/static
   sudo chown -R www-data:www-data /home/sar-bc/sites/data_tsnzv/media
   sudo chmod -R 755 /home/sar-bc/sites/data_tsnzv/media
   
   


