    #Установка Wordpress на Nginx.
    #Обновляем репозитории.
    sudo apt update && sudo apt upgrade -y
    #Устанавливаем веб-сервер, базу данных, SSH-сервер и программу для защиты от брутфорса по SSH.
    sudo apt install --no-install-recommends nginx mariadb-server php php-fpm php-zip php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc wget openssh-server fail2ban
    #Включаем веб-сервер.
    sudo systemctl start nginx
    Кладём в папку /etc/nginx/sites-enabled/ файл website.conf и редактируем его сообразно комментариям в этом файле.
    sudo cp website.conf /etc/nginx/sites-enabled/website.conf
    sudo nano /etc/nginx/sites-enabled/website.conf
    # У вас может быть версия php отличная от 7.4. Для просмотра вашей версии наберите ls /etc/php/ Соответственно, во всех последуюхих командах меняем 7.4 на вашу версию.
    sudo mv /etc/php/7.4/fpm/pool.d/www.conf /etc/php/7.4/fpm/pool.d/www.conf.old
    #Кладём в папку /etc/php/7.4/fpm/pool.d/ файл ubuntu-wp.conf
    sudo cp ubuntu-wp.conf /etc/php/7.4/fpm/pool.d/ubuntu-wp.conf
    # Перезапускаем службы
    sudo systemctl reload php7.4-fpm
    sudo systemctl restart php7.4-fpm
    sudo systemctl restart nginx
    sudo systemctl  status php7.4-fpm
    #Настраиваем базу данных. Пароль root, имя БД, имя пользователя БД и пароль от него выставляем по собственному усмотрению и записываем в надёжное место.
    sudo mysql -u root
   
      set password for 'root'@'localhost' = PASSWORD('password');
      create database wp_mysitedb;
      CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'samplepassword';
      GRANT ALL PRIVILEGES ON  wp_mysitedb. * TO 'wpuser'@'localhost';
      FLUSH PRIVILEGES;
      exit;
     #Скачиваем архив с Wordpress.
    cd /var/www/
    sudo wget https://ru.wordpress.org/latest-ru_RU.tar.gz
    sudo tar xfv latest-ru_RU.tar.gz 
    sudo chown -R www-data:www-data wordpress/
    sudo rm /etc/nginx/sites-enabled/default
    sudo systemctl reload nginx
    sudo systemctl restart nginx
    #Настройка SSL. Без SSL доступ к серверу будет по http, то есть без шифрования (Если только не использовать Tor, но об этом позже) Для настройки SSL объедините три сертификата (сам SSL-сертификат, корневой и промежуточный сертификаты) в один файл. Для этого создайте на ПК новый текстовый документ с именем your_domain.crt (your_domain — доменное имя сайта, который вы хотите защитить). Создать его можно при помощи блокнота или другого текстового редактора. Поочередно скопируйте и вставьте в созданный документ каждый сертификат. После вставки всех сертификатов файл должен иметь вид:

-----BEGIN CERTIFICATE-----
#Ваш сертификат#
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
#Промежуточный сертификат#
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
#Корневой сертификат#
-----END CERTIFICATE-----

#Обратите внимание: один сертификат идёт следом за другим, без пустых строк.
#Создайте файл your_domain.key и скопируйте в него содержание приватного ключа сертификата.
#Загрузите созданные файлы your_domain.crt и your_domain.key на сервер в директорию /etc/ssl/, а вместо конфигурационного файла website.conf кладём в папку /etc/nginx/sites-enabled/ файл website-ssl.conf и редактируем его сообразно комментариям в этом файле.
#Бонус: использование Tor:
#Tor позволяет разместить сайт с Wordpress в доменной зоне .onion. Использование Tor позволяет обойтись без использования публичного ip, записи DNS и SSL-сертификатов. Tor обеспечивает шифрование от клиента до сервера, аутентификацию сервера, адресацию и маршрутизацию.
#Установка Tor:
sudo apt install tor
#Настройка Tor: откроем файл /etc/tor/torrc и раскомментируем следующие строки:
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:80
#После чего перезапустим Tor:
sudo systemctl restart tor
#Далее узнаем присвоенный адрес в сети Tor
sudo cat /var/lib/tor/hidden_service/hostname

#Заходим из браузера по ip или url нашего сервера. В случае, если у сайта есть доменное имя, заходим по нему, в случае, если использовали Tor, заходим по .onion адоесу.

