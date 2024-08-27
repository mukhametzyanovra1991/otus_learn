# Групповые политики

## Цель:
Настравиать и котролировать инфраструкту с помощью GPO;  
Решать рутинные задачи с помощью Powershell;

## Описание/Пошаговая инструкция выполнения домашнего задания:
Пошаговая инструкция  
создайте в оснастке ad ou employes, manager, finance по одному пользователю  
создать GPO на каждый OU указанный выше который будет создать txt файл на рабочем столе employes.manager,finance соотвествено имени OU=файл  
вывести результат gpo в cmd под любым пользователем  
вывести информацию по любому пользователю через powershell  
заблокировать всех пользоватей в ou finance с помощью powershell  

## создайте в оснастке ad ou employes, manager, finance по одному пользователю
### create ou employes, manager, finance
New-ADOrganizationalUnit -Name "rustem.ru" -Path “DC=rustem,DC=ru” –Description “main OU” -PassThru  
$ou = @('employes', 'manager', 'finance')  
$ou | foreach {New-ADOrganizationalUnit -Name $_ -Path “OU=rustem.ru,DC=rustem,DC=ru” –Description “$_ OU” -PassThru}  
### create users
New-ADUser -Name "Иван Петров" -GivenName "Иван" -Surname "Петров" -SamAccountName "petrov_i" -UserPrincipalName "petrov_i@rustem.ru" -Path "OU=employes,OU=rustem.ru,DC=rustem,DC=ru" -AccountPassword(Read-Host -AsSecureString "Input Password") -Enabled $true

New-ADUser -Name "Сергей Смирнов" -GivenName "Сергей" -Surname "Смирнов" -SamAccountName "smirnov_s" -UserPrincipalName "smirnov_s@rustem.ru" -Path "OU=finance,OU=rustem.ru,DC=rustem,DC=ru" -AccountPassword(Read-Host -AsSecureString "Input Password") -Enabled $true

New-ADUser -Name "Виталий Иванов" -GivenName "Виталий" -Surname "Иванов" -SamAccountName "ivanov_v" -UserPrincipalName "ivanov_v@rustem.ru" -Path "OU=finance,OU=rustem.ru,DC=rustem,DC=ru" -AccountPassword(Read-Host -AsSecureString "Input Password") -Enabled $true

## создать GPO на каждый OU указанный выше который будет создать txt файл на рабочем столе employes.manager,finance соотвествено имени OU=файл
