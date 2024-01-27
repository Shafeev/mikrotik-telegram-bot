# mikrotik-telegram-bot
Mikrotik Telegram Bot

## Добавление скрипта в mikrotik
Запустите winBox и перейдите в скрипты System->Scripts

Добавьте скрипт c названием Telegram ![image](https://github.com/Shafeev/mikrotik-telegram-bot/assets/22525713/05bfe86e-d481-4a3c-bb56-7a424f95f799)

```
:delay 15
:global mtIdentity [/system identity get name];
:global botID "<you_bot_id>";
:global myChatID "<you_char_id>";
:local chatId 0;
:local messageId 0;

:local parse do={
:local startLoc ([:find $content $variable -1] + [:len $variable] + 2);
:local commaLoc ([:find $content "," $startLoc] - 1 + 1);
:local braceLoc ([:find $content "}" $startLoc] - 1 + 1);
:local endLoc $commaLoc;
:local startSymbol [:pick $content $startLoc]
:if ($braceLoc != 0 and ($commaLoc = 0 or $braceLoc < $commaLoc)) do={
:set endLoc $braceLoc;
};
:if ($startSymbol = "{") do={
:set endLoc ($braceLoc + 1);
};
:if ($quotas = true) do={
:set startLoc ($startLoc + 1);
:set endLoc ($endLoc - 1);
}
:if ($endLoc < $startLoc) do={
:set endLoc ($startLoc + 1);
};
:local message [:pick $content $startLoc $endLoc]
#:log info $message;
:return $message;
}

:while ( true ) do={
:do {
#:log info "https://api.telegram.org/$botID/getUpdates?offset=$messageId&limit=1&allowed_updates=message&timeout=60 10";
:tool fetch url=("https://api.telegram.org/$botID/getUpdates?offset=$messageId&limit=1&allowed_updates=message&timeout=60 10") dst-path="getUpdates";
:local content [/file get [/file find name=getUpdates] contents] ;
#:log info $content;
:if ([:len $content] > 30) do={
:set messageId ([$parse content=$content variable="update_id"] + 1)
:local message [$parse content=$content variable="text" quotas=true]
:local chat [$parse content=$content variable="chat"]
:local chatId [$parse content=$chat variable="id"]
:if (($chatId = $myChatID) and ([/system script find name=$message] != "")) do={
:system script run $message;
} else={
:tool fetch url=("https://api.telegram.org/$botID/sendmessage\?chat_id=$chatId&text=Unknown command: $message") keep-result=no
}
}
} on-error={}
};
```
