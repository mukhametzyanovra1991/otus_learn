# Парольная политика и логирование

## Цель:
Разграничивать парольную политику для услиление безопасности;
Производить поиск по логам информации по изменению пароля;

## Описание/Пошаговая инструкция выполнения домашнего задания:
Пошаговая инструкция  
создать парольные политики для OU employes, manager, finance на длину пароля 8, 16, 24  
вывести парольную политику для пользователя в ou manager с помощью Powershell  
настроить гпо по логированию  
создать gpo с рандомным именем, включить сбросить пароль пользователю в OU employes, даллее найти лог по изменению в остастке mmc и вывести резельтат по id лога в powershell  

## создать парольные политики для OU employes, manager, finance на длину пароля 8, 16, 24

## вывести парольную политику для пользователя в ou manager с помощью Powershell
PS C:\Users\Администратор.DC1> Get-ADUser -SearchBase "OU=manager,OU=rustem.ru,DC=rustem,DC=ru" -Filter * | Get-ADUserResultantPasswordPolicy  

AppliesTo                   : {CN=manager,OU=manager,OU=rustem.ru,DC=rustem,DC=ru}  
ComplexityEnabled           : True  
DistinguishedName           : CN=manager_pass,CN=Password Settings Container,CN=System,DC=rustem,DC=ru  
LockoutDuration             : 00:30:00  
LockoutObservationWindow    : 00:30:00  
LockoutThreshold            : 0  
MaxPasswordAge              : 90.00:00:00  
MinPasswordAge              : 1.00:00:00  
MinPasswordLength           : 16  
Name                        : manager_pass  
ObjectClass                 : msDS-PasswordSettings  
ObjectGUID                  : 10f1c6dd-d579-4be9-9c47-edd169f3f458  
PasswordHistoryCount        : 24  
Precedence                  : 1  
ReversibleEncryptionEnabled : False  

## настроить гпо по логированию
