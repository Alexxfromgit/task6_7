# task6_7

Общие требования к выполнению практических заданий
    1. Выполненное задание должно располагаться в отдельном репозитарии на github.com, т.е. если учетная запись на github называется ‘user’ и выполняется практическое задание с названием ‘task6_7’, то файлы задания должны быть доступны в репозитарии https://github.com/user/task6_7. 
    2. Список утвержденных для проверки репозитариев будет опубликован после ручной проверки заданий 2-го этапа
    3. Для проверки выполнения ДЗ будет использована свежеустановленне VM из этого образа - https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img
    4. Для выполнения ДЗ разрешается устанавливать любые дополнительные пакеты, помимо тех, что явно указаны в задании. В случае использования пакетов, которые не установлены в образе, необходимо предусмотреть их установку.
    5. Запуск скриптов во время проекта будет осуществляться от имени root
    6. Практические Задания по ЛК6-7 должны быть выполнены и загружены в соответствующие репозитарии до 23:59 19/04/18.


Задание  
Создание окружения
Создайте топологию из 2х ВМ как представлено на рис. 1 ниже


Рисунок 1 - Топология задания

VM1 должна иметь 3 сетевых интерфейса:
    • External - для подключения к Internet. Тип сети - не важен, однако подразумевается возможность подключения ко всем внешним сетям и Internet через этот интерфейс
    • Internal - для связи с VM2. Тип сети к которой подключен интерфейс Internal - “Host only”. К этой сети должны быть подключены лишь интерфейсы VM1 и VM2.
    • MGMT - для удаленного управления VM. Этот интерфейс должен иметь тип Bridged для подключения к VM со стороны хостовой системы
VM2 должна иметь 2 сетевых интерфейса:
    • Internal - для связи с VM1. Тип сети к которой подключен интерфейс Internal - “Host only”. К этой сети должны быть подключены лишь интерфейсы VM1 и VM2.
    • MGMT - для удаленного управления VM. Этот интерфейс должен иметь тип Bridged для подключения к VM со стороны хостовой системы
    • Выход в Internet c VM2 должен осуществляться через VM1

Задание
Создать скрипты конфигурации vm1.sh и vm2.sh которые выполнят сетевую конфигурацию соответствующих виртуальных машин, устанавливают и конфигурируют соответствующие веб-сервера. 
Часть 1 - Настройка сети
Для сетевой настройки виртуальных машин скрипты vm*.sh должны импортировать параметры из соответствующих vm*.config файлов. В этих файлах должны задаваться переменные соответствия сетевых интерфейсов VM и сетевой топологии на рисунке 1. Пример файла vm1.config для настройки VM1:
EXTERNAL_IF=”ens3”
INTERNAL_IF=”ens4”
MANAGEMENT_IF=”ens5”
VLAN=278
EXT_IP=”DHCP” или пара параметров (EXT_IP=172.16.1.1/24, EXT_GW=172.16.1.254)
INT_IP=10.0.0.1/24
VLAN_IP=YY.YY.YY.YY/24
NGINX_PORT=AAAA
APACHE_VLAN_IP=ZZ.ZZ.ZZ.ZZ
Пример файла vm2.config:
INTERNAL_IF=”ens3”
MANAGEMENT_IF=”ens4”
VLAN=278
APACHE_VLAN_IP=ZZ.ZZ.ZZ.ZZ/24
INT_IP=10.0.0.2/24
GW_IP=10.0.0.1

Каждый из скриптов должен выполнять следующие действия
    1. Настройку сетевых интерфейсов, согласно параметров, заданных в  конфигурационных файлах vm1.config и vm2.config. 
        a. Предусмотреть возможность настройки интерфейса External как с помощью статических параметров IP/Mask + внешний шлюз, так и по DHCP. 
    2. Настройку VLAN сабинтерфейсов, согласно параметров конфигурационных файлов vm1.config и vm2.config. 
        a. Название VLAN сабинтерфейса должно быть сгенерировано по шаблону <iface>.<VID>. Например для конфигурационных параметров
….
INTERNAL_IF=”ens4”
VLAN = 53
Название VLAN сабинтерфейса должно быть ens4.53
    3. Настройку маршрутизации там, где необходимо

Часть 2 - Настройка сервисов
Скрипты настройки виртуальных машин vm*.sh должны выполнить установку необходимых пакетов и последующую конфигурацию сервисов. Скрипт  vm1.sh должен выполнять следующие действия:
    1. Установку пакета nginx
    2. Генерацию ssl сертификатов
        a. Генерация корневого сертификата для VM1. Сертификат открытого ключа  должен находиться в /etc/ssl/certs/root-ca.crt
        b. Генерацию сертификата для веб-сервера (подписанного корневым сертификатом). Сертификат должен находиться в /etc/ssl/certs/web.crt. Сертификат должен быть сгенерирован на имя хоста (vm1) и иметь IP адрес External_IP в поле subjectAltName
    3. Настройку nginx сервера в качестве ssl termination proxy. При настройке необходимо выполнить следующие условия:
        a. Nginx должен быть настроен только для работы по протоколу https, возможность работы по протоколу http должна быть отключена.
        b. Для терминирования SSL использовать сертификаты, сгенерированные на шаге #2
        c. Nginx должен отдавать полную цепочку сертификатов.
        d. Nginx процесс должен принимать запросы только через интерфейс, указанный в переменной External из файла vm1.config. 
        e. Nginx должен принимать запросы на порту указанном в переменной NGINX_PORT из файла vm1.config. 
        f. Nginx должен перенаправлять http запросы на VM2. Адрес перенаправления запросов указан в переменной APACHE_VLAN_IP из файла vm1.config 
Скрипт  vm2.sh должен выполнять следующие действия:
    1. Установку пакета apache2
    2. Настройку веб-сервера apache согласно следующих требований:
        a. Apache процесс должен принимать запросы только через VLAN сабинтерфейс.

Дополнительные указания и требования

    1. Доступ в Internet с VM2 должен осуществляться через VM1. Шлюз для VM2 задается параметром GW_IP в файле vm2.config. 
    2. MANAGEMENT интерфейсы будут сконфигурированы автоматически на этапе проверки, т.е. скрипты vm*.sh не должны выполнять никаких настроек MANAGEMENT_IF интерфейсов. 
    3. Параметры сетевых интерфейсов в файлах vm*.config, должны экспортироваться соответствующими скриптами конфигурации vm*.sh.  
    4. Выполненные задания должны быть загружены в гитхаб репозитарий c названием ‘task6_7’
    5. В репозитарии ‘task6_7’ должны содержаться 4 файла:
        a. vm1.sh - файл конфигурации VM1.
        b. vm1.config - файл с параметрами сетевых интерфейсов VM1.
        c. vm2.sh - файл конфигурации VM2.
        d. vm2.config - файл с параметрами сетевых интерфейсов VM2.
Проверка результатов:
    • Для каждого запуска будут использоваться отдельные ВМ (ОС ubuntu xenial 16.04 server, образ). Т.е. считайте, что до вас никто ничего не настраивал. Скрипт будет запущен из-под суперпользователя (root). VM1 имеет доступ в интернет, VM2 - только в случае корректной настройки скриптами
    • На обе системы будет склонирован репозиторий с заданием (например https://github.com/user/task6_7, если репозиторий будет иметь другое имя, то задание будет автоматически помечено как невыполненное.
    • Скрипты и файлы с переменными (vm*.config и  vm*.sh) будут загружены на VM (т.е. на каждой вм доступны все файлы из репозитория task6_7).
    • Вначале будет выполнена конфигурация VM1, затем - VM2.
    • C хостовой системы будет выполнен запрос на веб-сервер nginx по протоколу https, сконфигурированный на VM1. В ответ должна прийти страница, настроенная на веб-сервер apache c VM2. Проверка валидности сертификата Nginx будет осуществлена на основе корневого сертификата из директории /etc/ssl/certs/root-ca.crt виртуальной машины VM1.

