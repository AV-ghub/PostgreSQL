<details><summary><h4><a href="https://github.com/AV-ghub/PostgreSQL-Cloud-Solutions/blob/main/Linux/CentOS/Intro/Installation/001%20Installation.md">Installation</a></h4></summary>

  ### Шаг 2: Создание виртуальной машины для CentOS

  * Запустите VirtualBox Менеджер и нажмите на кнопку «Создать»
  * Впишите имя машины
  * Укажите объем оперативной памяти
  * Оставьте выбранным пункт «Создать новый виртуальный жесткий диск»
  * Тип тоже не меняйте и оставьте VDI
  * Предпочтительный формат хранения — «динамический»
  * Размер для виртуального HDD выберите, исходя из доступного свободного места

  ### Шаг 3: Настройка виртуальной машины
  * Для входа в настройки нужно нажать правой кнопкой мыши по виртуальной машине и выбрать пункт «Настроить»
  * Во вкладке «Система» — «Процессор» можно увеличить количество процессоров до 2
  * Перейдя в «Дисплей», можете добавить некоторое количество МБ к видеопамяти и включить 3D-ускорение

  ### Шаг 4: Установка CentOS
  * Выделите кликом мыши виртуальную машину и нажмите на кнопку «Запустить»
  * После запуска VM нажмите на папку и через стандартный системный проводник укажите место, куда вы скачали образ ОС
  * Запустится установщик системы. При помощи стрелки вверх на клавиатуре выберите пункт «Install CentOS Linux 7» и нажмите Enter
  * Запустится графический установщик CentOS. Выберите ваш язык и его разновидность.
  * В окне с параметрами настройте:
  * Часовой пояс
  * Расположение установки
  * зайдите в меню с настройками, выделите виртуальный накопитель, который был создан вместе с виртуальной машиной, и нажмите «Готово»
  * Выбор программ
  * По умолчанию стоит минимальная установка, но она не имеет графического интерфейса. Вы можете выбрать, с какой средой будет установлена ОС: GNOME или KDE. Выбор зависит от ваших предпочтений, а мы рассмотрим инсталляцию с окружением KDE. После выбора оболочки в правой части окна появятся дополнения. Галочками можете отметить то, что хотели бы видеть в CentOS. По завершении выбора нажмите «Готово». ![](https://github.com/AV-ghub/PostgreSQL/blob/main/006%20Use%20cases/Installation/New%20cloud%20environment/CentOS%20based/Res/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20CentOS%203.jpg)
  * Нажмите на кнопку «Начать установку».
  * Во время установки (состояние отображается в нижней части окна как прогресс-бар) вам будет предложено придумать пароль root и создать пользователя.
  * Впишите пароль для прав root (суперпользователя) 2 раза и нажмите «Готово». Если пароль будет простым, кнопку «Готово» потребуется нажать дважды. Не забудьте сперва переключить раскладку клавиатуры на английский язык. Текущий язык можно увидеть в правом верхнем углу окна.
  * Впишите желаемые инициалы в поле «Полное имя». Строка «Имя пользователя» будет заполнена автоматически, но ее можно изменить вручную. При желании назначьте этого пользователя администратором, установив соответствующую галочку. Придумайте пароль для учетной записи и нажмите «Готово».
  * Дождитесь установки ОС и нажмите на кнопку «Завершить настройку».
  * Нажмите на кнопку «Перезагрузка».
  * Появится загрузчик GRUB, который по умолчанию через 5 секунд продолжит загрузку ОС. Можно сделать это вручную, не дожидаясь таймера, нажав на Enter.
  * Появится окно загрузки CentOS.
  * Снова отобразится окно с настройками. На этот раз нужно принять условия лицензионного соглашения и настроить сеть.
  * Чтобы включить интернет, нажмите на параметр «Сеть и имя узла». Кликните на регулятор, и он сдвинется вправо.  ![](https://github.com/AV-ghub/PostgreSQL/blob/main/006%20Use%20cases/Installation/New%20cloud%20environment/CentOS%20based/Res/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20CentOS%204.jpg)
  * Нажмите на кнопку «Завершить».
  * Вы попадете на экран входа в учетную запись. Кликните на нее.
  * Переключите раскладку клавиатуры, введите пароль и нажмите «Войти»

</details>

<details><summary><h4><a href="https://confluence.speechpro.com/pages/viewpage.action?pageId=165486262">Интернет. Прозрачный прокси-сервер (transparent proxy)</a></h4></summary>

  ```
  wget --no-check-certificate https://it-repo.speechpro.com/repository/raw-it-public/ca/stc_root.crt
  wget --no-check-certificate https://it-repo.speechpro.com/repository/raw-it-public/ca/stc_ica.crt
   
  sudo cp stc_{root,ica}.crt /etc/pki/ca-trust/source/anchors/
  sudo update-ca-trust extract
  ```

</details>

<details><summary><h4><a href="https://serverfault.com/questions/904304/could-not-resolve-host-mirrorlist-centos-org-centos-7">Could not resolve host: mirrorlist.centos.org Centos 7</a></h4></summary>

  
  From first of July 2024 on CentOS 7, please switch to Vault archive repositories:

  ```
  sudo nano /etc/yum.repos.d/CentOS-Base.repo
  ```
  
  copy/paste the following and mind your OS version. Change if needed. In this config is version 7.9.2009:
  
  ```
  [base]
  name=CentOS-$releasever - Base
  baseurl=http://vault.centos.org/7.9.2009/os/$basearch/
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
  
  [updates]
  name=CentOS-$releasever - Updates
  baseurl=http://vault.centos.org/7.9.2009/updates/$basearch/
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
  
  [extras]
  name=CentOS-$releasever - Extras
  baseurl=http://vault.centos.org/7.9.2009/extras/$basearch/
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
  
  [centosplus]
  name=CentOS-$releasever - Plus
  baseurl=http://vault.centos.org/7.9.2009/centosplus/$basearch/
  gpgcheck=1
  enabled=0
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
  ```

  If needed, do:
  
  ```
  yum clean all
  ```

</details>





























