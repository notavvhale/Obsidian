сброс ограничения 2-х часового с рудеска

![[Pasted image 20260325134418.png]]
1. Через Диспетчер задач закрываем, два процесса RuDesktop, тем самым закрывается программа и служба RuDesktop
2. Переходим в папку C:\Windows\ServiceProfiles\LocalService\AppData\Roaming\RuDesktop\config
3. В текстовом редакторе редактируем файл RuDesktop.toml  
    Строка с id = 'Ваш ID в системе RuDesktop' изменяем на id = '0'  
    Выходим с сохранением.
4. Перезагружаем компьютер