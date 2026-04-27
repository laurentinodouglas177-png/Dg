# Dg
Scannerdg7
#!/system/bin/sh
echo -e "\033[1;35m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo -e "      ⚡ DG7 SS INICIANDO ANÁLISE COMPLETA...⚡"
echo -e 
Y(){ printf "\033[%sm%s\033[0m\n" "$1" "$2"; }
K(){ echo "────────────────────────────────────────"; }

A=0

U(){
echo -n "[$1] "
Z=$(eval "$2" 2>/dev/null)

if [ -n "$Z" ] && echo "$Z" | grep -Eqi "$3"; then
Y 31 "ALERTA"
echo "  → $5"
echo "    Detalhe: $Z"
A=$((A+1))
else
Y 32 "OK"
echo "  → $4"
fi
}

################
# SYSTEM INFO
################

K
Y 35 "SYSTEM INFO"

echo "[uname]"
uname -a
echo ""

echo "[kernel]"
cat /proc/version
echo ""

U "disable magisk modules" 
"touch /data/adb/modules/.disable_modules && ls /data/adb/modules/.disable_modules" 
".disable_modules" 
"módulos magisk desarmados (vulnerável)" 
"falha ao desarmar módulos magisk"
U "check magisk zygisk" 
"magisk -v; magisk --sqlite "SELECT value FROM settings WHERE key='zygisk';"" 
"1" 
"zygisk ativo (ocultação detectada)" 
"zygisk inativo ou não encontrado"
U "kernelsu module bypass" 
"ls /data/adb/ksu/modules.img 2>/dev/null" 
"modules.img" 
"kernelsu detectado (possível ocultação)" 
"kernelsu não detectado no caminho padrão"
U "apatch safe mode trigger" 
"touch /data/adb/ap/safe_mode && ls /data/adb/ap/safe_mode" 
"safe_mode" 
"apatch forçado para modo seguro" 
"falha ao acessar diretório apatch"
U "su binary exposure" 
"which su && ls -l $(which su)" 
"/system/bin/su" 
"binário su exposto e vulnerável" 
"binário su oculto ou inexistente"
U "mount namespace check" 
"cat /proc/self/mounts | grep -E 'magisk|ksu|apatch'" 
"magisk" 
"rastros de montagem encontrados" 
"nenhum rastro de montagem detectado"


#                 ~ROOT~                  #

U "su -v" "su -v 2>/dev/null" "Magisk\|SuperSU\|KernelSU" "Nenhum gerenciador root detectado" "Gerenciador root detectado"

U "su -c id" "su -c id 2>/dev/null" "uid=0" "Sem execução root" "Executou comando como root"

U "which su" "which su 2>/dev/null" "/su\|/system\|/vendor" "Binário su não encontrado" "Binário su encontrado no sistema"

U "id" "id 2>/dev/null" "uid=0" "Usuário normal" "Usuário é ROOT"

U "getprop ro.build.tags" "getprop ro.build.tags 2>/dev/null" "test-keys" "ROM oficial (release-keys)" "ROM modificada (test-keys)"

K
Y 35 

U "ksu denylist files" \
"find /data -iname '*deny*' 2>/dev/null" \
"deny" \
"nenhuma denylist encontrada" \
"possível denylist detectada"

U "magisk denylist" \
"magisk --denylist ls 2>/dev/null" \
"." \
"denylist não acessível" \
"denylist do magisk ativa"

U "ksu policy traces" \
"logcat -d 2>/dev/null | grep -Ei 'su.*(granted|denied)|magisk.*(denylist|hide)|ksu.*(deny|allow|policy)'" \
"su.*(granted|denied)|magisk.*(denylist|hide)|ksu.*(deny|allow|policy)" \
"nenhuma política detectada" \
"atividade de política de root detectada"

U "su access logs" \
"logcat -d 2>/dev/null | grep -Ei 'granted|denied|su'" \
"granted|denied" \
"nenhum log de permissão" \
"permissões de root sendo usadas"

U "packages with root access" \
"dumpsys package 2>/dev/null | grep -Ei 'su|magisk|ksu'" \
"su|magisk|ksu" \
"nenhum app relacionado" \
"apps com possível acesso root"

U "zygisk deny traces" \
"cat /proc/\$(pidof zygote 2>/dev/null)/maps 2>/dev/null | grep -Ei 'deny|hide|zygisk'" \
"deny|hide|zygisk" \
"nenhum traço de ocultação" \
"possível ocultação ativa no zygote"

U "props suspicious" \
"getprop 2>/dev/null | grep -Ei 'ro.debuggable=1|ro.secure=0|ro.build.tags=test-keys|ro.boot.verifiedbootstate=orange|ro.boot.flash.locked=0'" \
"ro.debuggable=1|ro.secure=0|test-keys|verifiedbootstate=orange|flash.locked=0" \
"nenhuma prop suspeita" \
"propriedades suspeitas detectadas"

U "recent policy change" \
"ls -lt /data/adb 2>/dev/null" \
"." \
"nenhuma alteração recente" \
"arquivos modificados recentemente (possível mudança de política)"

U "recent adb top5" \
"ls -lt /data/adb 2>/dev/null | head -n 5" \
"." \
"nenhuma atividade relevante recente" \
"top 5 arquivos recentes em /data/adb detectados"


#                 ~TMP~                  #

U "tmp list" "ls -A /data/local/tmp 2>/dev/null" "." "tmp vazio ou inacessível" "Arquivos encontrados no tmp"

U "tmp executáveis reais" "find /data/local/tmp -type f -perm -111 ! -name '*.sh' 2>/dev/null" "." "Nenhum executável suspeito" "Executável suspeito no tmp"

U "tmp nomes críticos" "ls /data/local/tmp 2>/dev/null | grep -Eio 'frida|gum|inject|linjector'" "frida\|gum\|inject" "Nenhum nome crítico" "Arquivo crítico no tmp"

U "tmp binários grandes" "find /data/local/tmp -type f -size +10M 2>/dev/null" "." "Nenhum binário grande" "Binário grande suspeito"

U "tmp atividade recente" "find /data/local/tmp -type f -mmin -2 2>/dev/null" "." "Sem atividade recente" "Atividade recente suspeita"

U "tmp libs suspeitas" "find /data/local/tmp -type f -iname 'lib*.so' 2>/dev/null" "lib.*\.so" "Nenhuma lib suspeita" "Biblioteca carregável no tmp"

U "tmp scripts perigosos" "find /data/local/tmp -type f -iname '*.sh' -size +50k 2>/dev/null" "\.sh" "Nenhum script suspeito" "Script grande suspeito"

U "tmp permissões inseguras" "ls -ld /data/local/tmp 2>/dev/null" "rwxrwxrwx" "Permissão padrão" "Permissão 777 detectada"

U "tmp symlink suspeito" "find /data/local/tmp -type l 2>/dev/null" "." "Nenhum symlink suspeito" "Symlink detectado no tmp"

U "tmp execução em memória" "cat /proc/*/maps 2>/dev/null | grep -F '/data/local/tmp'" "/data/local/tmp" "Nenhuma execução em memória" "Execução via tmp detectada"

U "tmp processo ativo" "ps -A -o pid,args 2>/dev/null | grep -F '/data/local/tmp' | grep -v grep" "/data/local/tmp" "Nenhum processo ativo" "Processo executando via tmp"

#                 ~PROPRIEDADS                  #

U "ro.secure" "getprop ro.secure 2>/dev/null" "^0$" "Secure normal" "Sistema inseguro (ro.secure=0)"

U "ro.debuggable" "getprop ro.debuggable 2>/dev/null" "^1$" "Não debuggable" "Sistema debuggable"

U "adb root" "getprop service.adb.root 2>/dev/null" "^1$" "ADB normal" "ADB rodando como root"

U "build tags" "getprop ro.build.tags 2>/dev/null" "test-keys" "Build release (release-keys)" "Build test-keys"

U "boot state" "getprop ro.boot.verifiedbootstate 2>/dev/null" "orange\|red" "Boot verificado (green)" "Boot não verificado"

U "bootloader" "getprop ro.boot.flash.locked 2>/dev/null" "^0$" "Bootloader bloqueado" "Bootloader desbloqueado"

U "vbmeta" "getprop ro.boot.vbmeta.device_state 2>/dev/null" "unlocked" "vbmeta locked" "vbmeta unlocked"

#                 ~PARTIÇÕES~               # 

U "system rw" "mount 2>/dev/null | grep ' /system '" "rw," "system ro" "system montado como rw"

U "vendor rw" "mount 2>/dev/null | grep ' /vendor '" "rw," "vendor ro" "vendor montado como rw"

U "product rw" "mount 2>/dev/null | grep ' /product '" "rw," "product ro" "product montado como rw"

#                 ~BINARIOS SU~               #

35 "SU BIN"


for p in \
/system/bin/su \
/system/xbin/su \
/sbin/su \
/system/su \
/system/bin/.ext/.su \
/data/local/su \
/data/local/bin/su \
/data/local/xbin/su \
/su/bin/su \
/system/sbin/su \
/vendor/bin/su \
/system/app/Superuser.apk \
/data/adb/magisk \
/data/adb/ksu \
/data/adb/ap \
/cache/su \
/dev/com.koushikdutta.superuser.daemon
do
    ls -l "$p" 2>/dev/null
done

U "su xbin" "ls /system/xbin/su 2>/dev/null" "su$" "Não encontrado" "su em xbin"
U "su bin" "ls /system/bin/su 2>/dev/null" "su$" "Não encontrado" "su em bin"
U "su sbin" "ls /sbin/su 2>/dev/null" "su$" "Não encontrado" "su em sbin"
U "su vendor" "ls /vendor/bin/su 2>/dev/null" "su$" "Não encontrado" "su em vendor"

U "/system/bin" "ls /system/bin 2>/dev/null | grep -E '^su$|^magisk$|^busybox$|^ksu$|^apd$'" "su|magisk|busybox|ksu|apd" "Nenhum binário suspeito" "Resultado encontrado em /system/bin"

U "/system/xbin" "ls /system/xbin 2>/dev/null | grep -E '^su$|^magisk$|^busybox$|^ksu$|^apd$'" "su|magisk|busybox|ksu|apd" "Nenhum binário suspeito" "Resultado encontrado em /system/xbin"

U "/system_ext/bin" "ls /system_ext/bin 2>/dev/null | grep -E '^su$|^magisk$|^busybox$|^ksu$'" "su|magisk|busybox|ksu" "Nenhum binário suspeito" "Resultado encontrado em system_ext"

U "/product/bin" "ls /product/bin 2>/dev/null | grep -E '^su$|^magisk$|^busybox$|^ksu$'" "su|magisk|busybox|ksu" "Nenhum binário suspeito" "Resultado encontrado em product"

U "/vendor/bin" "ls /vendor/bin 2>/dev/null | grep -E '^su$|^magisk$|^busybox$|^ksu$'" "su|magisk|busybox|ksu" "Nenhum binário suspeito" "Resultado encontrado em vendor"

U "/odm/bin" "ls /odm/bin 2>/dev/null | grep -E '^su$|^magisk$|^busybox$|^ksu$'" "su|magisk|busybox|ksu" "Nenhum binário suspeito" "Resultado encontrado em odm"

echo ""
echo "[scan su global]"

find /system /vendor /sbin /data -type f -name su 2>/dev/null
find / -type f -name su 2>/dev/null

#                 ~MAGISK~               #

K
Y 35 "MAGISK"

U "magisk pkg" "pm list packages 2>/dev/null | grep -Ei 'magisk'" "magisk" "Nenhum app magisk" "Magisk instalado (app)"

U "magisk pasta" "ls -d /data/adb/magisk 2>/dev/null" "/data/adb/magisk" "Sem pasta magisk" "Pasta magisk existe"

U "magisk modules" "ls -A /data/adb/modules 2>/dev/null" "." "Sem módulos" "Módulos magisk encontrados"

U "magisk daemon" "ps -A 2>/dev/null | grep -i magisk | grep -v grep" "magisk" "Nenhum daemon" "Daemon magisk rodando"

U "magisk zygisk" "ls /data/adb 2>/dev/null | grep -i zygisk" "zygisk" "Sem zygisk" "Zygisk detectado"

U "magisk arquivos" "find /data/adb -iname '*magisk*' 2>/dev/null" "magisk" "Nenhum arquivo magisk" "Arquivos magisk encontrados"

#                ~MAGISK MOUNT~               #

K

ls -l /sbin/.magisk 2>/dev/null
ls -l /dev/.magisk 2>/dev/null

U "magisk dir base" "ls /data/adb 2>/dev/null | grep -i '^magisk$'" "magisk" "Nenhum diretório Magisk" "Diretório Magisk detectado"

U "magisk módulos" "ls -A /data/adb/modules 2>/dev/null" "." "Nenhum módulo" "Módulos Magisk detectados"

U "magisk módulos ativos" "find /data/adb/modules -type f -name 'module.prop' 2>/dev/null" "module.prop" "Nenhum módulo ativo" "Módulo ativo detectado"

U "magisk módulos desativados" "find /data/adb/modules -type f -name 'disable' 2>/dev/null" "disable" "Nenhum módulo desativado" "Módulo desativado detectado"

U "magisk post-fs-data" "find /data/adb -type f -name 'post-fs-data.sh' 2>/dev/null" "post-fs-data.sh" "Nenhum script" "Script post-fs-data detectado"

U "magisk service" "find /data/adb -type f -name 'service.sh' 2>/dev/null" "service.sh" "Nenhum service" "Service script detectado"

U "magisk mount" "cat /proc/self/mounts 2>/dev/null | grep -i magisk" "magisk" "Nenhum mount Magisk" "Magisk mount detectado"

U "magisk overlay system" "cat /proc/self/mounts 2>/dev/null | grep -E ' /system | /vendor ' | grep -i overlay" "overlay" "Sem overlay" "Overlay suspeito (Magisk)"

U "magisk espelho" "ls -ld /sbin/.magisk 2>/dev/null" ".magisk" "Nenhum espelho Magisk" "Espelho Magisk detectado"

U "magisk binários" "find /sbin /system/bin /system/xbin -type f -iname '*magisk*' 2>/dev/null" "magisk" "Nenhum binário Magisk" "Binário Magisk detectado"

U "magisk maps" "cat /proc/*/maps 2>/dev/null | grep -i magisk" "magisk" "Nenhum Magisk na memória" "Magisk carregado na memória"

U "magisk init rc" "grep -i magisk /init.rc 2>/dev/null" "magisk" "Nenhuma referência no init" "Referência Magisk no init"

K
Y 35 "MAGISK TMP"

U "tmp arquivos" "ls /data/local/tmp 2>/dev/null | grep -Ei 'magisk'" "magisk" "Nenhum magisk no tmp" "Magisk detectado no tmp"

U "tmp find magisk" "find /data/local/tmp -type f 2>/dev/null | grep -Ei 'magisk'" "magisk" "Nenhum arquivo magisk" "Arquivo Magisk no tmp"

U "tmp bin executável" "find /data/local/tmp -type f -perm -111 2>/dev/null | grep -Ei 'magisk'" "magisk" "Nenhum executável" "Binário Magisk executável no tmp"

U "tmp scripts" "find /data/local/tmp -type f -iname '*.sh' 2>/dev/null | xargs grep -i magisk 2>/dev/null" "magisk" "Nenhum script magisk" "Script Magisk no tmp"

U "tmp libs" "find /data/local/tmp -type f -iname '*.so' 2>/dev/null | grep -Ei 'magisk'" "magisk" "Nenhuma lib magisk" "Lib Magisk no tmp"

U "tmp execução" "ps -A 2>/dev/null | grep -i '/data/local/tmp' | grep -Ei 'magisk'" "magisk" "Magisk não executando do tmp" "Magisk executando via tmp"

U "tmp maps" "cat /proc/*/maps 2>/dev/null | grep -i '/data/local/tmp' | grep -Ei 'magisk'" "magisk" "Nenhum magisk carregado" "Magisk carregado via tmp"

K
Y 35 "MAGISK MEMORIA"

U "magisk maps" "cat /proc/*/maps 2>/dev/null | grep -i magisk" "magisk" "Nenhum magisk na memória" "Magisk carregado na memória"

U "zygisk maps" "cat /proc/*/maps 2>/dev/null | grep -i zygisk" "zygisk" "Nenhum zygisk" "Zygisk ativo"

U "zygote hook" "cat /proc/*/maps 2>/dev/null | grep -Ei 'zygote|app_process' | grep -Ei 'magisk|zygisk'" "zygisk" "Sem hook" "Hook Zygisk detectado"

K
Y 35 "MAGISK MODULES"

U "modules dir" "[ -d /data/adb/modules ] && echo found" "found" "Sem modules" "Diretório modules existe"

U "modules conteúdo" "ls -A /data/adb/modules 2>/dev/null" "." "Sem módulos" "Módulos encontrados"

U "module.prop" "find /data/adb/modules -type f -name module.prop 2>/dev/null" "module.prop" "Nenhum module.prop" "Módulos reais detectados"

U "magisk módulo" "ls /data/adb/modules 2>/dev/null | grep -Ei 'magisk'" "magisk" "Nenhum módulo magisk" "Módulo Magisk detectado"

U "zygisk módulo" "ls /data/adb/modules 2>/dev/null | grep -Ei 'zygisk'" "zygisk" "Sem zygisk" "Zygisk detectado"

U "modules scripts" "find /data/adb/modules -type f \\( -name 'service.sh' -o -name 'post-fs-data.sh' \\) 2>/dev/null" "service.sh|post-fs-data" "Nenhum script" "Scripts de módulo detectados"

K
Y 35 "MAGISK ADB"

U "adb base" "[ -d /data/adb ] && echo found" "found" "Sem /data/adb" "Diretório /data/adb existe"

U "magisk pasta" "[ -d /data/adb/magisk ] && echo found" "found" "Sem pasta magisk" "Magisk detectado"

U "magisk bin adb" "ls /data/adb 2>/dev/null | grep -Ei 'magisk'" "magisk" "Nenhum arquivo magisk" "Arquivos Magisk em /data/adb"

U "magisk daemon files" "find /data/adb -type f 2>/dev/null | grep -Ei 'magiskd'" "magiskd" "Nenhum daemon" "magiskd encontrado"

U "magisk scripts adb" "find /data/adb -type f \\( -name 'service.sh' -o -name 'post-fs-data.sh' \\) 2>/dev/null" "service.sh|post-fs-data" "Nenhum script" "Scripts Magisk detectados"

K
Y 35 "MAGISK BINARIOS"

U "magisk bin path" "which magisk 2>/dev/null" "magisk" "Magisk não no PATH" "Magisk no PATH"

U "magisk bin system" "find /system /vendor /sbin -type f 2>/dev/null | grep -Ei 'magisk'" "magisk" "Nenhum binário sistema" "Binário Magisk no sistema"

U "magisk bin adb" "find /data/adb -type f 2>/dev/null | grep -Ei 'magisk'" "magisk" "Nenhum binário em /data/adb" "Binário Magisk em /data/adb"

U "magisk bin global" "find / -type f 2>/dev/null | grep -Ei 'magisk'" "magisk" "Nenhum binário global" "Binário Magisk encontrado globalmente"

U "magisk exec" "magisk -v 2>/dev/null" "." "Magisk não executável" "Magisk executando"

U "magisk permissões" "find /data/adb /system /vendor -type f -perm -111 2>/dev/null | grep -Ei 'magisk'" "magisk" "Nenhum executável" "Executável Magisk detectado"

#                 ~KERNELSU~               #

K
Y 35 "KERNELSU"

U "ksu pkg" "pm list packages 2>/dev/null | grep -Ei 'ksu'" "ksu" "Nenhum ksu" "KernelSU instalado"

U "ksu pasta" "[ -d /data/adb/ksu ] && echo found" "found" "KernelSU não encontrado" "KernelSU detectado"

U "ksu bin pasta" "[ -d /data/adb/ksu/bin ] && echo found" "found" "bin KernelSU não encontrado" "bin KernelSU detectado"

U "ksud daemon file" "ls /data/adb/ksu/bin 2>/dev/null | grep -E '^ksud$'" "ksud" "daemon ksud não encontrado" "daemon KernelSU detectado"

U "ksu su file" "ls /data/adb/ksu/bin 2>/dev/null | grep -E '^su$'" "su" "su ksu não encontrado" "su KernelSU detectado"

U "ksu modules" "ls -A /data/adb/ksu/modules 2>/dev/null" "." "sem módulos KernelSU" "módulos KernelSU detectados"

U "ksu modules_update" "[ -d /data/adb/ksu/modules_update ] && echo found" "found" "sem modules_update" "modules_update KernelSU detectado"

U "ksu service.d" "[ -d /data/adb/ksu/service.d ] && echo found" "found" "sem scripts service.d" "scripts service.d KernelSU detectados"

U "ksu post-fs-data" "[ -d /data/adb/ksu/post-fs-data.d ] && echo found" "found" "sem scripts post-fs-data" "scripts post-fs-data KernelSU detectados"

U "ksu processo" "ps -A 2>/dev/null | grep -i ksu | grep -v grep" "ksu" "Nenhum processo ksu" "Processo ksu rodando"


U "system xbin su" "ls /system/xbin/su 2>/dev/null" "su$" "su não encontrado em xbin" "su encontrado em xbin"

U "system bin su" "ls /system/bin/su 2>/dev/null" "su$" "su não encontrado em bin" "su encontrado em bin"


U "superuser apk" "ls /system/app 2>/dev/null | grep -E '^Superuser'" "Superuser" "Superuser apk não encontrado" "Superuser apk detectado"

U "supersu app" "ls /system/app 2>/dev/null | grep -E '^SuperSU'" "SuperSU" "SuperSU não encontrado" "SuperSU detectado"


U "kallsyms root" "cat /proc/kallsyms 2>/dev/null | grep -Ei 'ksu|magisk|apatch'" "ksu|magisk|apatch" "nenhum root no kallsyms" "root detectado no kernel"

U "kernel modules" "cat /proc/modules 2>/dev/null | grep -Ei 'ksu|magisk|apatch'" "ksu|magisk|apatch" "nenhum módulo suspeito" "módulo root no kernel"

#                 ~APATCH~               #

K
Y 35 "APATCH"

U "apatch pkg" "pm list packages 2>/dev/null | grep -Ei 'apatch'" "apatch" "Nenhum pacote APatch" "Aplicativo APatch instalado"

U "apatch dir1" "[ -d /data/adb/ap ] && echo found" "found" "Diretório /data/adb/ap não existe" "Diretório APatch encontrado"

U "apatch dir2" "[ -d /data/adb/apatch ] && echo found" "found" "Diretório /data/adb/apatch não existe" "Diretório APatch encontrado"

U "apatch daemon" "ps -A 2>/dev/null | awk '{print \$NF}' | grep -x apd" "apd" "Daemon APatch não encontrado" "Daemon APatch rodando"

U "apatch dir base" "ls /data/adb 2>/dev/null | grep -Ei '^apatch$|^ap$'" "apatch|ap" "Nenhum diretório APatch" "Diretório APatch detectado"

U "apatch módulos dir" "ls /data/adb/modules 2>/dev/null | grep -i apatch" "apatch" "Nenhum módulo APatch" "Módulo APatch detectado"

U "apatch módulos ativos" "find /data/adb/modules -type f -iname '*apatch*' 2>/dev/null" "apatch" "Nenhum módulo ativo APatch" "Arquivo de módulo APatch detectado"

U "apatch scripts init" "grep -Ri apatch /data/adb 2>/dev/null | grep -E 'post-fs-data.sh|service.sh'" "apatch" "Nenhum script APatch" "Script APatch detectado"

U "apatch mounts diretos" "cat /proc/self/mounts 2>/dev/null | grep -i apatch" "apatch" "Nenhum mount APatch" "Mount APatch detectado"

U "apatch overlay real" \
"cat /proc/self/mounts 2>/dev/null | grep overlay | grep -E 'upperdir=|workdir=' | grep -E '/system|/vendor|/product'" \
"upperdir=|workdir=" \
"Sem overlay suspeito" \
"Overlay suspeito (root)"

U "apatch mounts globais" "cat /proc/*/mounts 2>/dev/null | grep -i apatch" "apatch" "Nenhum mount global" "Mount APatch em outro processo"

U "apatch binários" "find /system /vendor /data -type f -iname '*apatch*' 2>/dev/null" "apatch" "Nenhum binário APatch" "Binário APatch detectado"

U "apatch memória" "cat /proc/*/maps 2>/dev/null | grep -i apatch" "apatch" "Nenhum APatch na memória" "APatch carregado na memória"

U "apatch propriedades" "getprop 2>/dev/null | grep -i apatch" "apatch" "Nenhuma propriedade APatch" "Propriedade APatch detectada"

K
Y 35 "APATCH MODULES"

U "apatch módulo dir" "ls /data/adb/modules 2>/dev/null | grep -Ei 'apatch|ap'" "apatch|ap" "Nenhum módulo APatch" "Módulo APatch detectado"

U "apatch module.prop" "grep -Ri apatch /data/adb/modules 2>/dev/null" "apatch" "Nenhum module.prop APatch" "APatch em module.prop"

U "apatch arquivos módulo" "find /data/adb/modules -type f -iname '*apatch*' 2>/dev/null" "apatch" "Nenhum arquivo APatch" "Arquivos APatch no módulo"

U "apatch bin módulo" "find /data/adb/modules -type f -iname '*ap*' 2>/dev/null | grep -Ei 'apd|apatch'" "apd|apatch" "Nenhum binário APatch" "Binário APatch no módulo"

U "apatch overlay system" "find /data/adb/modules -type d -path '*/system/*' 2>/dev/null | grep -Ei 'apatch|ap'" "apatch|ap" "Sem overlay APatch" "Overlay APatch detectado"

U "apatch scripts módulo" "grep -Ri apatch /data/adb/modules 2>/dev/null | grep -E 'service.sh|post-fs-data.sh'" "apatch" "Nenhum script APatch" "Script APatch detectado"

U "apatch symlink módulo" "find /data/adb/modules -type l 2>/dev/null | grep -Ei 'apatch|ap'" "apatch|ap" "Nenhum symlink APatch" "Symlink APatch detectado"

U "apatch mount ativo" "cat /proc/self/mounts 2>/dev/null | grep -Ei 'apatch|/data/adb/modules'" "apatch|modules" "Sem mount APatch" "Mount APatch detectado"

U "apatch maps módulo" "cat /proc/*/maps 2>/dev/null | grep -Ei 'apatch|apd'" "apatch|apd" "APatch não carregado" "APatch carregado (módulo)"

K
Y 35 "APATCH MANAGER"

U "apatch manager pkg exato" "pm list packages 2>/dev/null | grep -E '^package:.*apatch'" "apatch" "Nenhum APatch Manager" "APatch Manager instalado"

U "apatch manager variantes" "pm list packages 2>/dev/null | grep -Ei 'apatch|apatchmanager|ap.manager|apatch.app'" "apatch|manager" "Nenhuma variante APatch" "Variante APatch detectada"

U "apatch manager path" \
"for p in \$(pm list packages 2>/dev/null | grep -Ei 'apatch|apatchmanager' | cut -d: -f2); do pm path \$p; done" \
"package:" \
"Sem APK APatch" \
"APK APatch detectado"

U "apatch manager data dir" "ls /data/data 2>/dev/null | grep -Ei 'apatch|apatchmanager'" "apatch" "Sem diretório APatch" "Diretório do app APatch detectado"

U "apatch manager cache" "ls /data/data/*apatch*/cache 2>/dev/null" "." "Sem cache APatch" "Cache do APatch encontrado"

U "apatch manager libs" "ls /data/app 2>/dev/null | grep -Ei 'apatch'" "apatch" "Nenhuma lib/APK APatch" "APK APatch em /data/app"

U "apatch manager running" "ps -A 2>/dev/null | grep -Ei 'apatch' | grep -v grep" "apatch" "APatch não em execução" "APatch em execução"

K
Y 35 "APATCH BINARIOS"

U "apatch bin /data/adb/ap" "find /data/adb/ap -type f 2>/dev/null | grep -Ei 'apd|apatch'" "apd|apatch" "Nenhum binário em /data/adb/ap" "Binário APatch em /data/adb/ap"

U "apatch bin /data/adb/apatch" "find /data/adb/apatch -type f 2>/dev/null | grep -Ei 'apd|apatch'" "apd|apatch" "Nenhum binário em /data/adb/apatch" "Binário APatch em /data/adb/apatch"

U "apatch bin modules" "find /data/adb/modules -type f 2>/dev/null | grep -Ei 'apd|apatch'" "apd|apatch" "Nenhum binário em modules" "Binário APatch em modules"

U "apatch bin system" \
"find /system/bin /system/xbin /vendor/bin /sbin -type f 2>/dev/null | grep -E '/apatch$|/apd$'" \
"apatch|apd" \
"Nenhum binário APatch" \
"Binário APatch detectado"

U "apatch bin global" "find / -type f 2>/dev/null | grep -Ei 'apd|apatch'" "apd|apatch" "Nenhum binário global" "Binário APatch encontrado globalmente"

U "apatch exec apd" "which apd 2>/dev/null" "apd" "apd não no PATH" "apd no PATH"

U "apatch execução direta" "apd -v 2>/dev/null" "." "apd não executável" "apd executando"

U "apatch permissões" "find /data/adb /system /vendor -type f -perm -111 2>/dev/null | grep -Ei 'apd|apatch'" "apd|apatch" "Nenhum executável APatch" "Executável APatch detectado"

U "apatch tmp bin" "find /data/local/tmp -type f 2>/dev/null | grep -Ei 'apd|apatch'" "apd|apatch" "Nenhum binário em tmp" "Binário APatch em tmp"

U "apatch maps bin" "cat /proc/*/maps 2>/dev/null | grep -Ei 'apd|apatch'" "apd|apatch" "Binário não carregado" "Binário APatch carregado"

#                 ~FRAMEWORKS~               #

K
Y 35 "FRAMEWORKS"


U "xposed pkg" "pm list packages 2>/dev/null | grep -Ei 'xposed'" "xposed" "Nenhum xposed" "Xposed instalado"

U "lsposed pkg" "pm list packages 2>/dev/null | grep -Ei 'lsposed'" "lsposed" "Nenhum lsposed" "LSPosed instalado"

U "riru pkg" "pm list packages 2>/dev/null | grep -Ei 'riru'" "riru" "Nenhum riru" "Riru instalado"


U "xposed dir" "ls /data/data 2>/dev/null | grep -Ei '^de\.robv\.android\.xposed'" "xposed" "Nenhum diretório xposed" "Diretório xposed detectado"

U "lsposed módulo" "ls /data/adb/modules 2>/dev/null | grep -Ei '^lsposed'" "lsposed" "Nenhum LSPosed" "LSPosed nos módulos"

U "riru módulo" "ls /data/adb/modules 2>/dev/null | grep -Ei '^riru'" "riru" "Nenhum Riru" "Riru nos módulos"


U "xposed files" "find /data /system -type f -iname '*xposed*' 2>/dev/null" "xposed" "Nenhum arquivo xposed" "Arquivo xposed detectado"

U "lsposed files" "find /data -type f -iname '*lsposed*' 2>/dev/null" "lsposed" "Nenhum arquivo lsposed" "Arquivo lsposed detectado"

U "riru files" "find /data -type f -iname '*riru*' 2>/dev/null" "riru" "Nenhum arquivo riru" "Arquivo riru detectado"


U "xposed tmp" "ls /data/local/tmp 2>/dev/null | grep -Ei 'xposed'" "xposed" "Nenhum xposed no tmp" "Xposed no tmp detectado"

U "riru tmp" "ls /data/local/tmp 2>/dev/null | grep -Ei 'riru'" "riru" "Nenhum riru no tmp" "Riru no tmp detectado"

U "xposed maps" "cat /proc/*/maps 2>/dev/null | grep -i xposed" "xposed" "Nenhum xposed na memória" "Xposed carregado na memória"

U "lsposed maps" "cat /proc/*/maps 2>/dev/null | grep -i lsposed" "lsposed" "Nenhum lsposed na memória" "LSPosed carregado na memória"

U "riru maps" "cat /proc/*/maps 2>/dev/null | grep -i riru" "riru" "Nenhum riru na memória" "Riru carregado na memória"


U "zygisk maps" "cat /proc/*/maps 2>/dev/null | grep -i zygisk" "zygisk" "Nenhum zygisk" "Zygisk ativo"

U "zygote hook" "cat /proc/*/maps 2>/dev/null | grep -Ei 'app_process|zygote' | grep -Ei 'xposed|riru|lsposed|zygisk'" "zygote" "Nenhum hook zygote" "Hook no zygote detectado"


U "framework proc" "ps -A 2>/dev/null | grep -Ei 'xposed|riru|lsposed|zygisk' | grep -v grep" "xposed|riru|lsposed|zygisk" "Nenhum processo" "Framework ativo em execução"

#                 ~BUSYBOX~               #

K
Y 35 "BUSYBOX"

U "busybox bin (path)" "which busybox 2>/dev/null" "busybox" "BusyBox não encontrado no PATH" "BusyBox no PATH"

U "busybox system bin" "ls /system/bin/busybox 2>/dev/null" "busybox" "Sem busybox em /system/bin" "BusyBox em /system/bin"

U "busybox xbin" "ls /system/xbin/busybox 2>/dev/null" "busybox" "Sem busybox em /system/xbin" "BusyBox em /system/xbin"

U "busybox vendor" "ls /vendor/bin/busybox 2>/dev/null" "busybox" "Sem busybox em /vendor/bin" "BusyBox em /vendor/bin"

U "busybox data" "find /data -type f -name busybox 2>/dev/null" "busybox" "Nenhum busybox em /data" "BusyBox encontrado em /data"

U "busybox global" "find /system /vendor /sbin /data -type f -name busybox 2>/dev/null" "busybox" "Nenhum busybox global" "BusyBox encontrado no sistema"

U "busybox execução" "busybox 2>/dev/null" "BusyBox" "BusyBox não executável" "BusyBox executando"

U "busybox versão" "busybox | head -n 1 2>/dev/null" "BusyBox" "Sem versão" "Versão BusyBox detectada"

U "busybox applets (lista)" "busybox --list 2>/dev/null" "ls|sh|mount|grep" "Sem applets" "Applets BusyBox disponíveis"

U "busybox applets path" "busybox --list-full 2>/dev/null" "/" "Sem applets path" "Applets com path detectados"

U "busybox links" "ls -l /system/bin 2>/dev/null | grep busybox" "busybox" "Sem links BusyBox" "Links BusyBox encontrados"

U "busybox tmp" "ls /data/local/tmp 2>/dev/null | grep -i busybox" "busybox" "Nenhum busybox no tmp" "BusyBox no tmp detectado"

U "busybox maps" "cat /proc/*/maps 2>/dev/null | grep -i busybox" "busybox" "BusyBox não carregado" "BusyBox carregado na memória"

U "busybox proc" "ps -A 2>/dev/null | grep -i busybox | grep -v grep" "busybox" "Nenhum processo BusyBox" "BusyBox em execução"

K
Y 35 "BUSYBOX MODULES"

U "busybox módulo dir" "ls /data/adb/modules 2>/dev/null | grep -Ei 'busybox'" "busybox" "Nenhum módulo BusyBox" "Módulo BusyBox detectado"

U "busybox module.prop" "find /data/adb/modules -type f -iname 'module.prop' 2>/dev/null | xargs grep -i busybox 2>/dev/null" "busybox" "Nenhum module.prop BusyBox" "BusyBox em module.prop"

U "busybox arquivos módulo" "find /data/adb/modules -type f -iname '*busybox*' 2>/dev/null" "busybox" "Nenhum arquivo BusyBox" "Arquivos BusyBox no módulo"

U "busybox bin módulo" "find /data/adb/modules -type f -name busybox 2>/dev/null" "busybox" "Nenhum binário BusyBox" "Binário BusyBox no módulo"

U "busybox system overlay" "find /data/adb/modules -type d -path '*/system/*' 2>/dev/null | grep -i busybox" "busybox" "Sem overlay BusyBox" "Overlay BusyBox detectado"

U "busybox scripts módulo" "find /data/adb/modules -type f \\( -name 'service.sh' -o -name 'post-fs-data.sh' \\) 2>/dev/null | xargs grep -i busybox 2>/dev/null" "busybox" "Nenhum script BusyBox" "Script BusyBox detectado"

U "busybox links módulo" "find /data/adb/modules -type l 2>/dev/null | grep -i busybox" "busybox" "Nenhum symlink BusyBox" "Symlink BusyBox detectado"

U "busybox mount ativo" "cat /proc/self/mounts 2>/dev/null | grep -i busybox" "busybox" "BusyBox não montado" "BusyBox montado via módulo"

U "busybox maps módulo" "cat /proc/*/maps 2>/dev/null | grep -i busybox" "busybox" "BusyBox não carregado" "BusyBox carregado (módulo)"

#                 ~XPOSED~               #

K
Y 35 "XPOSED BINARIOS"

U "xposed bin system" "find /system /vendor /sbin -type f 2>/dev/null | grep -Ei 'xposed|lsposed|riru'" "xposed|lsposed|riru" "Nenhum binário sistema" "Binário Xposed detectado"

U "xposed bin global" "find / -type f 2>/dev/null | grep -Ei 'xposed|lsposed|riru'" "xposed|lsposed|riru" "Nenhum binário global" "Binário Xposed encontrado"

U "executáveis xposed" "find / -type f -perm -111 2>/dev/null | grep -Ei 'xposed|lsposed|riru'" "xposed|lsposed|riru" "Nenhum executável" "Executável Xposed detectado"

U "libs xposed" "find /system /vendor /data -type f -iname '*.so' 2>/dev/null | grep -Ei 'xposed|lsposed|riru'" "xposed|lsposed|riru" "Nenhuma lib" "Lib Xposed detectada"

K
Y 35 "XPOSED TMP"

U "tmp arquivos" "ls /data/local/tmp 2>/dev/null | grep -Ei 'xposed|lsposed|riru'" "xposed|lsposed|riru" "Nada no tmp" "Xposed no tmp"

U "tmp find" "find /data/local/tmp -type f 2>/dev/null | grep -Ei 'xposed|lsposed|riru'" "xposed|lsposed|riru" "Nenhum arquivo" "Arquivo Xposed no tmp"

U "tmp executável" "find /data/local/tmp -type f -perm -111 2>/dev/null | grep -Ei 'xposed|lsposed|riru'" "xposed|lsposed|riru" "Nenhum executável" "Executável Xposed no tmp"

U "tmp scripts" "find /data/local/tmp -type f -iname '*.sh' 2>/dev/null | xargs grep -Ei 'xposed|lsposed|riru' 2>/dev/null" "xposed|lsposed|riru" "Nenhum script" "Script Xposed no tmp"

U "tmp maps" "cat /proc/*/maps 2>/dev/null | grep '/data/local/tmp' | grep -Ei 'xposed|lsposed|riru'" "xposed|lsposed|riru" "Nada carregado" "Xposed carregado via tmp"

K
Y 35 "XPOSED MODULES"

U "modules base" "[ -d /data/adb/modules ] && echo found" "found" "Sem modules" "Modules existem"

U "modules conteúdo" "ls -A /data/adb/modules 2>/dev/null" "." "Sem módulos" "Módulos encontrados"

U "lsposed módulo" "ls /data/adb/modules 2>/dev/null | grep -Ei 'lsposed'" "lsposed" "Nenhum LSPosed" "LSPosed ativo"

U "riru módulo" "ls /data/adb/modules 2>/dev/null | grep -Ei 'riru'" "riru" "Nenhum Riru" "Riru ativo"

U "module.prop" "find /data/adb/modules -type f -name module.prop 2>/dev/null | xargs grep -Ei 'xposed|lsposed|riru' 2>/dev/null" "xposed|lsposed|riru" "Nenhum módulo Xposed" "Módulo Xposed detectado"

K
Y 35 "XPOSED PROCESSO"

U "processo xposed" "ps -A 2>/dev/null | grep -Ei 'xposed|lsposed|riru'" "xposed|lsposed|riru" "Nenhum processo" "Processo Xposed ativo"

U "zygote modificado" "ps -A 2>/dev/null | grep zygote" "zygote" "Zygote não encontrado" "Zygote ativo"

U "app_process hook" "ps -A 2>/dev/null | grep app_process" "app_process" "Sem app_process" "app_process ativo"

K
Y 35 "XPOSED MEMORIA"

U "maps xposed" "cat /proc/*/maps 2>/dev/null | grep -Ei 'xposed'" "xposed" "Nenhum xposed" "Xposed na memória"

U "maps lsposed" "cat /proc/*/maps 2>/dev/null | grep -Ei 'lsposed'" "lsposed" "Nenhum lsposed" "LSPosed na memória"

U "maps riru" "cat /proc/*/maps 2>/dev/null | grep -Ei 'riru'" "riru" "Nenhum riru" "Riru na memória"

U "zygote hook" "cat /proc/*/maps 2>/dev/null | grep -Ei 'zygote|app_process' | grep -Ei 'xposed|lsposed|riru'" "xposed|lsposed|riru" "Sem hook" "Hook Xposed no Zygote"

K
Y 35 

U "xprivacylua" "pm list packages | grep -i xprivacylua" "xprivacylua" "Não encontrado" "XPrivacyLua detectado"
U "disableflagsecure" "pm list packages | grep -i flagsecure" "flagsecure" "Não encontrado" "Disable Flag Secure detectado"
U "hidemyapplist" "pm list packages | grep -i hidemyapplist" "hidemyapplist" "Não encontrado" "Hide My Applist detectado"
U "rootcloak" "pm list packages | grep -i rootcloak" "rootcloak" "Não encontrado" "RootCloak detectado"
U "gravitybox" "pm list packages | grep -i gravitybox" "gravitybox" "Não encontrado" "GravityBox detectado"
U "luckypatcher" "pm list packages | grep -i luckypatcher" "luckypatcher" "Não encontrado" "Lucky Patcher Xposed detectado"
U "pixelify" "pm list packages | grep -i pixelify" "pixelify" "Não encontrado" "Pixelify detectado"
U "bootloaderspoofer" "pm list packages | grep -i bootloaderspoofer" "bootloaderspoofer" "Não encontrado" "BootloaderSpoofer detectado"

#                 ~HIDEN~               #

K

U "shamiko" "ls /data/adb/modules 2>/dev/null | grep -i shamiko" "shamiko" "Shamiko não encontrado" "Shamiko detectado"
U "roothide" "ls /data/adb/modules 2>/dev/null | grep -Ei 'roothide|hide'" "hide" "Nenhum root hide" "Root hide detectado"
U "apatch hide" "ls /data/adb/modules 2>/dev/null | grep -Ei 'apatchhide|apatch_hide'" "apatch" "APatch hide não encontrado" "APatch hide detectado"


U "hide arquivos adb" "find /data/adb -type f 2>/dev/null | grep -Ei 'shamiko|hide|deny|mask'" "hide" "Nenhum arquivo hide" "Arquivos de ocultação detectados"

U "hide scripts" "find /data/adb -type f \\( -name 'service.sh' -o -name 'post-fs-data.sh' \\) 2>/dev/null | xargs grep -Ei 'hide|shamiko|deny' 2>/dev/null" "hide" "Nenhum script hide" "Script de ocultação detectado"


U "hide tmp nomes" "ls /data/local/tmp 2>/dev/null | grep -Ei 'hide|mask|cloak|shamiko'" "hide" "Nada no tmp" "Ocultação no tmp"

U "hide tmp arquivos" "find /data/local/tmp -type f 2>/dev/null | grep -Ei 'hide|mask|cloak|shamiko'" "hide" "Nenhum arquivo" "Arquivo hide no tmp"

U "hide tmp execução" "ps -A 2>/dev/null | grep '/data/local/tmp' | grep -Ei 'hide|mask|cloak'" "hide" "Nada executando" "Ocultação executando via tmp"


U "root sockets" "cat /proc/net/unix 2>/dev/null | grep -Ei 'magisk|ksu|apatch'" "magisk|ksu|apatch" "Nenhum socket root" "Socket root detectado"


U "play integrity fix" "ls /data/adb/modules 2>/dev/null | grep -Ei 'playintegrity|play_integrity'" "play" "Play Integrity não encontrado" "Play Integrity Fix detectado"

U "safetynet fix" "ls /data/adb/modules 2>/dev/null | grep -Ei 'safetynet'" "safetynet" "SafetyNet não encontrado" "SafetyNet Fix detectado"

U "integrity box" "ls /data/adb/modules 2>/dev/null | grep -Ei 'integritybox'" "integrity" "Integrity Box não encontrado" "Integrity Box detectado"


U "integrity arquivos" "find /data/adb/modules -type f 2>/dev/null | grep -Ei 'integrity|safetynet'" "integrity" "Nenhum arquivo" "Arquivos de bypass detectados"


U "props config" "ls /data/adb/modules 2>/dev/null | grep -Ei 'props|magiskhideprops'" "props" "Props config não encontrado" "Props config detectado"

U "device spoof" "ls /data/adb/modules 2>/dev/null | grep -Ei 'device|spoof'" "device" "Device spoof não encontrado" "Device spoof detectado"

U "bootloader spoof" "ls /data/adb/modules 2>/dev/null | grep -Ei 'bootloader|spoofer'" "bootloader" "Bootloader spoof não encontrado" "Bootloader spoof detectado"


U "fingerprint" "getprop ro.build.fingerprint" "userdebug|test-keys" "Fingerprint normal" "Fingerprint suspeita"

U "model spoof" "getprop ro.product.model" "." "Sem info" "Modelo pode estar spoofado"

U "brand spoof" "getprop ro.product.brand" "." "Sem info" "Brand pode estar spoofada"


U "denylist módulos" "ls /data/adb/modules 2>/dev/null | grep -Ei 'denylist|deny'" "deny" "Nenhum denylist" "Denylist detectada"

U "apatch denylist" "ls /data/adb/modules 2>/dev/null | grep -Ei 'apatchdeny'" "apatchdeny" "APatch deny não encontrado" "APatch deny detectado"

U "denylist runtime" "magisk --denylist status 2>/dev/null" "enabled" "Denylist desativada" "Denylist ativa"


U "zygisk módulo" "ls /data/adb/modules 2>/dev/null | grep -i zygisk" "zygisk" "Sem zygisk" "Zygisk detectado"

U "riru core" "ls /data/adb/modules 2>/dev/null | grep -i riru" "riru" "Riru não encontrado" "Riru detectado"

U "lsposed módulo" "ls /data/adb/modules 2>/dev/null | grep -i lsposed" "lsposed" "LSPosed não encontrado" "LSPosed detectado"


U "hide maps" "cat /proc/*/maps 2>/dev/null | grep -Ei 'shamiko|hide|deny|mask'" "hide" "Nada na memória" "Ocultação carregada na memória"

U "integrity maps" "cat /proc/*/maps 2>/dev/null | grep -Ei 'integrity|safetynet'" "integrity" "Nada na memória" "Bypass na memória"

U "zygote hide hook" "cat /proc/*/maps 2>/dev/null | grep -Ei 'zygote|app_process' | grep -Ei 'hide|shamiko|deny'" "hide" "Sem hook" "Hook de ocultação no zygote"


U "hide processos" "ps -A 2>/dev/null | grep -Ei 'shamiko|hide|deny|mask'" "hide" "Nenhum processo" "Processo de ocultação ativo"

#                 ~SHAMIKO~               #

K
Y 35 "SHAMIKO"

U "shamiko módulo" "ls /data/adb/modules 2>/dev/null | grep -i shamiko" "shamiko" "Shamiko não encontrado" "Shamiko detectado"

U "shamiko arquivos" "find /data/adb -type f 2>/dev/null | grep -i shamiko" "shamiko" "Nenhum arquivo Shamiko" "Arquivos Shamiko detectados"

U "shamiko prop" "find /data/adb/modules -type f -name module.prop 2>/dev/null | xargs grep -i shamiko 2>/dev/null" "shamiko" "Nenhum module.prop" "Módulo Shamiko detectado (prop)"

U "shamiko scripts" "find /data/adb -type f \\( -name 'service.sh' -o -name 'post-fs-data.sh' \\) 2>/dev/null | xargs grep -i shamiko 2>/dev/null" "shamiko" "Nenhum script" "Script Shamiko detectado"

U "shamiko tmp" "find /data/local/tmp -type f 2>/dev/null | grep -i shamiko" "shamiko" "Nada no tmp" "Shamiko no tmp"

U "shamiko tmp exec" "ps -A 2>/dev/null | grep '/data/local/tmp' | grep -i shamiko" "shamiko" "Não executando no tmp" "Shamiko executando via tmp"

U "shamiko maps" "cat /proc/*/maps 2>/dev/null | grep -i shamiko" "shamiko" "Nenhum shamiko na memória" "Shamiko carregado na memória"

U "shamiko zygote" "cat /proc/*/maps 2>/dev/null | grep -Ei 'zygote|app_process' | grep -i shamiko" "shamiko" "Sem hook" "Shamiko hook no zygote"

U "shamiko processos" "ps -A 2>/dev/null | grep -i shamiko" "shamiko" "Nenhum processo" "Processo Shamiko ativo"

U "shamiko ocultação genérica" "cat /proc/*/maps 2>/dev/null | grep -Ei 'hide|mask|deny' | grep -vi system" "hide" "Nenhuma ocultação" "Possível ocultação ativa"

K
Y 35 "SHAMIKO"

U "shamiko módulo" "ls /data/adb/modules 2>/dev/null | grep -i shamiko" "shamiko" "Shamiko não encontrado" "Shamiko detectado"

U "shamiko módulos deep" "find /data/adb/modules -maxdepth 2 -type d 2>/dev/null | grep -i shamiko" "shamiko" "Nenhum diretório" "Diretório Shamiko detectado"

U "shamiko module.prop" "find /data/adb/modules -type f -name module.prop 2>/dev/null | xargs grep -i shamiko 2>/dev/null" "shamiko" "Nenhum module.prop" "Módulo Shamiko detectado"

U "shamiko arquivos adb" "find /data/adb -type f 2>/dev/null | grep -i shamiko" "shamiko" "Nenhum arquivo" "Arquivo Shamiko detectado"

U "shamiko binários" "find /data/adb /system /vendor -type f -perm -111 2>/dev/null | grep -i shamiko" "shamiko" "Nenhum binário" "Binário Shamiko detectado"

U "shamiko libs" "find /data/adb /system /vendor -type f -iname '*.so' 2>/dev/null | grep -i shamiko" "shamiko" "Nenhuma lib" "Lib Shamiko detectada"

U "shamiko scripts" "find /data/adb -type f \\( -name 'service.sh' -o -name 'post-fs-data.sh' \\) 2>/dev/null | xargs grep -i shamiko 2>/dev/null" "shamiko" "Nenhum script" "Script Shamiko detectado"

U "shamiko tmp" "find /data/local/tmp -type f 2>/dev/null | grep -i shamiko" "shamiko" "Nada no tmp" "Shamiko no tmp"

U "shamiko tmp exec" "ps -A 2>/dev/null | grep '/data/local/tmp' | grep -i shamiko" "shamiko" "Não executando no tmp" "Shamiko executando via tmp"

U "shamiko maps" "cat /proc/*/maps 2>/dev/null | grep -i shamiko" "shamiko" "Nenhum shamiko na memória" "Shamiko carregado na memória"

U "shamiko zygote" "cat /proc/*/maps 2>/dev/null | grep -Ei 'zygote|app_process' | grep -i shamiko" "shamiko" "Sem hook" "Hook Shamiko no zygote"

U "shamiko processos" "ps -A 2>/dev/null | grep -i shamiko" "shamiko" "Nenhum processo" "Processo Shamiko ativo"

U "shamiko ocultação genérica" "cat /proc/*/maps 2>/dev/null | grep -Ei 'hide|mask|deny' | grep -vi system" "hide" "Nenhuma ocultação" "Possível ocultação ativa"

#             ~KERNELSU~             #

K
Y 35 "KSU HIDE"

U "ksu hide módulo" "ls /data/adb/modules 2>/dev/null | grep -Ei 'ksuhide|kernelsuhide'" "hide" "KernelSU Hide não encontrado" "KernelSU Hide detectado"

U "ksu stealth módulo" "ls /data/adb/modules 2>/dev/null | grep -Ei 'stealth|ksustealth'" "stealth" "KernelSU Stealth não encontrado" "KernelSU Stealth detectado"

U "ksu denylist módulo" "ls /data/adb/modules 2>/dev/null | grep -Ei 'ksudeny|ksu_deny'" "deny" "KernelSU DenyList não encontrado" "KernelSU DenyList detectado"

U "ksu zygisk módulo" "ls /data/adb/modules 2>/dev/null | grep -Ei 'zygiskksu|ksuzygisk'" "zygisk" "Zygisk KSU não encontrado" "Zygisk for KernelSU detectado"

U "ksu módulos deep" "find /data/adb/modules -maxdepth 2 -type d 2>/dev/null | grep -Ei 'ksu|kernelsu'" "ksu" "Nenhum diretório" "Diretório KernelSU detectado"

U "ksu module.prop" "find /data/adb/modules -type f -name module.prop 2>/dev/null | xargs grep -Ei 'ksu|kernelsu|hide|deny' 2>/dev/null" "ksu" "Nenhum module.prop" "Módulo KSU detectado"

U "ksu arquivos adb" "find /data/adb -type f 2>/dev/null | grep -Ei 'ksu|kernelsu|hide|deny'" "ksu" "Nenhum arquivo" "Arquivos KernelSU detectados"

U "ksu binários" "find /data/adb /system /vendor -type f -perm -111 2>/dev/null | grep -Ei 'ksu|kernelsu'" "ksu" "Nenhum binário" "Binário KernelSU detectado"

U "ksu libs" \
"find /data/adb -maxdepth 3 -type f 2>/dev/null | grep -Ei 'ksu|kernelsu'" \
"ksu|kernelsu" \
"Nenhum traço KernelSU" \
"Arquivos KernelSU detectados"

U "ksu scripts" "find /data/adb -type f \\( -name 'service.sh' -o -name 'post-fs-data.sh' \\) 2>/dev/null | xargs grep -Ei 'ksu|kernelsu|hide|deny' 2>/dev/null" "ksu" "Nenhum script" "Script KernelSU detectado"

U "ksu tmp arquivos" "find /data/local/tmp -type f 2>/dev/null | grep -Ei 'ksu|kernelsu|hide|deny'" "ksu" "Nada no tmp" "KernelSU no tmp"

U "ksu tmp execução" "ps -A 2>/dev/null | grep '/data/local/tmp' | grep -Ei 'ksu|kernelsu|hide'" "ksu" "Nada executando" "KernelSU executando via tmp"

U "ksu maps" "cat /proc/*/maps 2>/dev/null | grep -Ei 'ksu|kernelsu'" "ksu" "Nenhum KSU na memória" "KernelSU carregado na memória"

U "ksu zygote hook" "cat /proc/*/maps 2>/dev/null | grep -Ei 'zygote|app_process' | grep -Ei 'ksu|kernelsu|zygisk'" "ksu" "Sem hook" "Hook KernelSU no zygote"

U "ksu processos" "ps -A 2>/dev/null | grep -Ei 'ksu|kernelsu'" "ksu" "Nenhum processo" "Processo KernelSU ativo"

U "ksu ocultação genérica" "cat /proc/*/maps 2>/dev/null | grep -Ei 'hide|mask|deny' | grep -vi system" "hide" "Nenhuma ocultação" "Possível ocultação ativa"

U "magiskboot trace detect" 
"ls -R /data 2>/dev/null | grep -i 'magiskboot'" 
"magiskboot" 
"binário de patch magiskboot encontrado no sistema" 
"nenhum rastro de ferramenta de patch magiskboot"
U "ksud binary check" 
"which ksud || find /data -name ksud 2>/dev/null" 
"ksud" 
"ferramenta de patch KernelSU (ksud) detectada" 
"ksud não encontrado"
U "kernel image mismatch" 
"cat /proc/version | grep -E 'KSU|Magisk|Patch'" 
"KSU" 
"kernel modificado via patch detectado (KernelSU)" 
"assinatura de kernel limpa ou padrão"
U "boot partition integrity" 
"ls -l /dev/block/by-name/boot*" 
"boot" 
"partição de boot acessível para análise de dump" 
"partição de boot protegida ou oculta"
U "init_boot modification" 
"ls -l /dev/block/by-name/init_boot*" 
"init_boot" 
"rastros de patch em init_boot detectados" 
"init_boot parece original"
U "kmi version spoofing" 
"getprop | grep -i 'kmi'" 
"kmi" 
"tentativa de spoofing de versão KMI detectada" 
"propriedades de KMI consistentes"

#             ~OVERLAYFS~            #

K
Y 35 "OVERLAYFS"

U "overlay básico" \
"cat /proc/mounts 2>/dev/null | grep overlay | grep -E 'upperdir=|workdir=' | grep -E '/system|/vendor|/product' | grep -vE '/apex|/linkerconfig'" \
"upperdir=|workdir=" \
"Nenhum overlay suspeito" \
"OverlayFS em partição crítica"

U "overlay self mounts" \
"cat /proc/self/mounts 2>/dev/null | grep overlay | grep -E 'upperdir=|workdir='" \
"upperdir=|workdir=" \
"Nenhum overlay local" \
"OverlayFS ativo no processo"

U "overlay global mounts" \
"cat /proc/*/mounts 2>/dev/null | grep overlay | grep -E 'upperdir=|workdir='" \
"upperdir=|workdir=" \
"Nenhum overlay global" \
"OverlayFS ativo em processos"

U "overlay system" \
"cat /proc/mounts 2>/dev/null | grep overlay | grep -E 'upperdir=|workdir=' | grep -E ' /system '" \
"upperdir=|workdir=" \
"System limpo" \
"Overlay real em /system"

U "overlay vendor" \
"cat /proc/mounts 2>/dev/null | grep overlay | grep -E 'upperdir=|workdir=' | grep -E ' /vendor '" \
"upperdir=|workdir=" \
"Vendor limpo" \
"Overlay real em /vendor"

U "overlay product" \
"cat /proc/mounts 2>/dev/null | grep overlay | grep -E 'upperdir=|workdir=' | grep -E ' /product '" \
"upperdir=|workdir=" \
"Product limpo" \
"Overlay real em /product"

U "overlay adb" "cat /proc/mounts 2>/dev/null | grep -Ei 'overlay' | grep -Ei '/data/adb'" "adb" "Sem ligação adb" "Overlay ligado ao /data/adb"

U "overlay upperdir" "cat /proc/mounts 2>/dev/null | grep -i overlay | grep -i upperdir" "upperdir" "Sem upperdir" "Overlay com upperdir (modificação ativa)"

U "overlay workdir" "cat /proc/mounts 2>/dev/null | grep -i overlay | grep -i workdir" "workdir" "Sem workdir" "Overlay com workdir"

U "overlay magisk indício" "cat /proc/mounts 2>/dev/null | grep -i overlay | grep -Ei 'magisk|adb'" "magisk|adb" "Sem indício magisk" "Overlay ligado ao Magisk"

U "overlay ksu indício" "cat /proc/mounts 2>/dev/null | grep -i overlay | grep -Ei 'ksu|kernelsu'" "ksu" "Sem indício KSU" "Overlay ligado ao KernelSU"

U "overlay apatch indício" "cat /proc/mounts 2>/dev/null | grep -i overlay | grep -Ei 'apatch|apd'" "apatch" "Sem indício APatch" "Overlay ligado ao APatch"

K
Y 35 "SELINUX"

U "getenforce" "getenforce 2>/dev/null" "Permissive" "SELinux Enforcing" "SELinux Permissive"

U "sestatus" "sestatus 2>/dev/null" "permissive" "SELinux ativo" "SELinux permissive detectado"

U "kernel enforce" "cat /sys/fs/selinux/enforce 2>/dev/null" "0" "Kernel enforcing" "Kernel permissive"

U "selinux fs" "ls -d /sys/fs/selinux 2>/dev/null" "/sys/fs/selinux" "SELinux FS ausente" "SELinux FS presente"

U "init contexto" "ls -Z /proc/1 2>/dev/null" "u:r:init" "Contexto alterado" "Contexto init válido"

U "processos untrusted" "ps -Z 2>/dev/null | grep -v 'u:r:'" "u:" "Contextos válidos" "Processos com contexto suspeito"

U "selinux permissive processos" "ps -Z 2>/dev/null | grep permissive" "permissive" "Nenhum processo permissive" "Processo permissive detectado"

U "audit logs" "dmesg 2>/dev/null | grep -i avc" "avc" "Sem logs AVC" "Eventos SELinux detectados"

U "selinux disable prop" "getprop 2>/dev/null | grep -i selinux" "permissive" "Sem alteração em props" "Propriedade SELinux suspeita"

U "enforce cmdline" "cat /proc/cmdline 2>/dev/null | grep -Ei 'selinux=0|enforcing=0'" "selinux=0|enforcing=0" "Boot normal" "SELinux desativado no boot"

#             ~INJEÇÃO~                #

K
Y 35 "INJECTIONS"

U "frida proc" "ps -A 2>/dev/null | grep -i frida" "frida" "Nenhum frida" "Frida rodando"

U "frida server" "ps -A 2>/dev/null | grep -Ei 'frida-server'" "frida" "Nenhum frida-server" "Frida server ativo"

U "frida portas" "netstat -an 2>/dev/null | grep -Ei '27042|27043'" "27042|27043" "Nenhuma porta frida" "Porta Frida aberta"

U "frida arquivos" "find / -type f 2>/dev/null | grep -Ei 'frida'" "frida" "Nenhum frida" "Arquivo Frida detectado"

U "frida tmp" "find /data/local/tmp -type f 2>/dev/null | grep -Ei 'frida'" "frida" "Nada no tmp" "Frida no tmp"

U "maps frida" "cat /proc/*/maps 2>/dev/null | grep -Ei 'frida'" "frida" "Nenhum frida na memória" "Frida injetado na memória"

U "maps libs suspeitas" "cat /proc/*/maps 2>/dev/null | grep -Ei '\\.so' | grep -Ei 'inject|hook|substrate|xposed|frida'" "inject|hook|substrate|xposed|frida" "Nenhuma lib suspeita" "Lib suspeita carregada"

U "tmp execução" "ps -A 2>/dev/null | grep '/data/local/tmp'" "/data/local/tmp" "Nada executando" "Execução via tmp detectada"

U "ptrace status" "cat /proc/*/status 2>/dev/null | grep -i tracerpid" "TracerPid:\t0" "Sem ptrace" "Processo sendo rastreado"

U "zygote maps" "cat /proc/*/maps 2>/dev/null | grep -Ei 'zygote|app_process' | grep -Ei 'magisk|ksu|apatch|frida|xposed'" "magisk|ksu|apatch|frida|xposed" "Zygote limpo" "Injeção no zygote"

U "app_process check" "ps -A 2>/dev/null | grep app_process" "app_process" "Sem app_process" "app_process ativo"

U "ld preload" "cat /proc/*/environ 2>/dev/null | grep -i LD_PRELOAD" "LD_PRELOAD" "Sem preload" "LD_PRELOAD detectado"

U "debuggable apps" "getprop ro.debuggable 2>/dev/null" "1" "Sistema não debug" "Sistema debug habilitado"

#           ~HOOK | INLINE             #

K
Y 35 "HOOK | INLINE"

U "maps rwx" "cat /proc/self/maps 2>/dev/null | grep -E 'rwxp'" "rwxp" "Sem memória RWX" "Memória RWX suspeita (injeção)"

U "maps anon exec" \
"cat /proc/self/maps 2>/dev/null | grep -E 'r-xp' | grep '00:00 0' | grep -vE '/system|/apex|/vendor|/product|/data/app|/dev/ashmem|/memfd:'" \
"r-xp" \
"Execução normal" \
"Execução anônima fora do padrão detectada"

U "memfd exec" "cat /proc/self/maps 2>/dev/null | grep -i memfd" "memfd" "Nenhum memfd" "Execução via memfd (stealth)"

U "deleted libs" "cat /proc/self/maps 2>/dev/null | grep '(deleted)'" "(deleted)" "Sem libs deletadas" "Lib apagada ainda em memória"

U "proc fd exec" "ls -l /proc/self/fd 2>/dev/null | grep memfd" "memfd" "Sem fd suspeito" "FD memfd suspeito"

U "linker bypass" \
"cat /proc/self/maps 2>/dev/null | grep -Ei 'linker' | grep -vE '/system/bin/linker|/apex/.*/linker'" \
"linker" \
"Linker normal" \
"Linker fora do padrão detectado"

U "plt hook check" "readelf -r /proc/self/exe 2>/dev/null | grep -i 'jmp'" "jmp" "Sem indício" "Possível hook PLT/GOT"

U "inline hook scan" "hexdump -C /proc/self/exe 2>/dev/null | grep -Ei 'ff e0|ff e1|ff e2'" "ff e0" "Sem inline hook" "Possível JMP hook detectado"

U "syscall hook" "cat /proc/kallsyms 2>/dev/null | grep sys_call_table" "sys_call_table" "Tabela syscall protegida" "Possível hook syscall"

U "kprobe detect" "cat /sys/kernel/debug/kprobes/list 2>/dev/null" "0" "Sem kprobe" "Kprobe ativo (hook kernel)"

U "uprobes detect" "cat /sys/kernel/debug/tracing/uprobe_events 2>/dev/null" "p:" "Sem uprobes" "Uprobe ativo"

U "ptrace global" "cat /proc/*/status 2>/dev/null | grep -v 'TracerPid:\t0'" "TracerPid" "Sem ptrace" "Processo sendo rastreado"

U "seccomp check" "cat /proc/self/status 2>/dev/null | grep Seccomp" "0" "Sem filtro seccomp" "Seccomp ativo"

U "maps sem path" \
"cat /proc/self/maps 2>/dev/null | grep -E 'rwxp|r-xp' | grep '00:00 0' | grep -vE '\\[anon:|/dev/ashmem|/memfd:'" \
"rwxp|r-xp" \
"Sem região anônima suspeita" \
"Memória anônima executável suspeita"

U "elf tmp exec" "ls -l /data/local/tmp 2>/dev/null | grep '\\.so'" ".so" "Nenhuma lib tmp" "Lib executável no tmp"

U "app lib spoof" "ls /data/app/*/lib 2>/dev/null | grep -Ei 'hook|inject'" "hook" "Libs normais" "Lib suspeita em app"

U "zygote inline" "cat /proc/*/maps 2>/dev/null | grep -Ei 'zygote' | grep -Ei 'rwx|memfd'" "rwx|memfd" "Zygote limpo" "Zygote comprometido"

U "stack exec" "cat /proc/self/maps 2>/dev/null | grep '\\[stack\\]' | grep 'rwx'" "rwx" "Stack seguro" "Stack executável (exploit)"

U "vdso hook" \
"cat /proc/self/maps 2>/dev/null | grep vdso | grep -v '\\[vdso\\]'" \
"vdso" \
"vdso ok" \
"vdso fora do padrão detectado"

U "libsubstrate" \
"cat /proc/self/maps | grep libsubstrate.so" \
"libsubstrate.so" \
"não encontrada" \
"libsubstrate detectada"

U "libsubstrate-dvm" \
"cat /proc/self/maps | grep libsubstrate-dvm.so" \
"libsubstrate-dvm.so" \
"não encontrada" \
"libsubstrate-dvm detectada"

U "libsubstrate-dalvik" \
"cat /proc/self/maps | grep libsubstrate-dalvik.so" \
"libsubstrate-dalvik.so" \
"não encontrada" \
"libsubstrate-dalvik detectada"

U "libsandhook" \
"cat /proc/self/maps | grep libsandhook.so" \
"libsandhook.so" \
"não encontrada" \
"libsandhook detectada"

U "libwhale" \
"cat /proc/self/maps | grep libwhale.so" \
"libwhale.so" \
"não encontrada" \
"libwhale detectada"

U "libyahfa" \
"cat /proc/self/maps | grep libyahfa.so" \
"libyahfa.so" \
"não encontrada" \
"libyahfa detectada"

U "libepic" \
"cat /proc/self/maps | grep libepic.so" \
"libepic.so" \
"não encontrada" \
"libepic detectada"

U "libandhook" \
"cat /proc/self/maps | grep libandhook.so" \
"libandhook.so" \
"não encontrada" \
"libandhook detectada"

U "libxposed_art" \
"cat /proc/self/maps | grep libxposed_art.so" \
"libxposed_art.so" \
"não encontrada" \
"libxposed_art detectada"

U "liblsposed" \
"cat /proc/self/maps | grep liblsposed.so" \
"liblsposed.so" \
"não encontrada" \
"liblsposed detectada"

U "libriru" \
"cat /proc/self/maps | grep libriru.so" \
"libriru.so" \
"não encontrada" \
"libriru detectada"

U "libzygisk" \
"cat /proc/self/maps | grep libzygisk.so" \
"libzygisk.so" \
"não encontrada" \
"libzygisk detectada"

U "libdobby" \
"cat /proc/self/maps | grep libdobby.so" \
"libdobby.so" \
"não encontrada" \
"libdobby detectada"

U "libhookzz" \
"cat /proc/self/maps | grep libhookzz.so" \
"libhookzz.so" \
"não encontrada" \
"libhookzz detectada"

U "libshadowhook" \
"cat /proc/self/maps | grep libshadowhook.so" \
"libshadowhook.so" \
"não encontrada" \
"libshadowhook detectada"

U "libbhook" \
"cat /proc/self/maps | grep libbhook.so" \
"libbhook.so" \
"não encontrada" \
"libbhook detectada"

#         ~PATH SUSPEITOS~             #

K
Y 35 "HOOK PATHS"

U "frida tmp" \
"ls /data/local/tmp 2>/dev/null | grep -Ei 'frida|gum|gadget'" \
"frida" \
"nada encontrado" \
"frida encontrado em /data/local/tmp"

U "tmp exec geral" \
"ls /data/local/tmp 2>/dev/null" \
"." \
"tmp vazio" \
"Arquivos presentes no tmp (possível injeção)"

U "frida dev" \
"ls /dev 2>/dev/null | grep -i frida" \
"frida" \
"nenhum frida em /dev" \
"frida escondido em /dev"

U "frida cache" \
"ls /data/data/*/cache 2>/dev/null | grep -Ei 'frida|gum|gadget'" \
"frida" \
"nenhum cache suspeito" \
"arquivo frida em cache"

U "global lib scan" \
"find /data /system /vendor 2>/dev/null | grep -Ei '\\.so' | grep -Ei 'hook|inject|frida|substrate|xposed'" \
"hook|inject|frida|substrate|xposed" \
"nenhuma lib suspeita" \
"lib suspeita encontrada no sistema"

U "lsposed module lib" \
"ls /data/adb/modules/lsposed 2>/dev/null | grep liblsposed.so" \
"liblsposed.so" \
"não encontrado" \
"liblsposed encontrado em módulo"

U "riru module lib" \
"ls /data/adb/modules 2>/dev/null | grep -i riru" \
"riru" \
"riru não encontrado" \
"riru presente nos módulos"

U "zygisk libs" \
"find /data/adb 2>/dev/null | grep -Ei 'zygisk|zygote'" \
"zygisk" \
"nenhum zygisk" \
"zygisk encontrado"

U "magisk libs" \
"find /data/adb 2>/dev/null | grep -Ei 'magisk'" \
"magisk" \
"nenhum magisk" \
"magisk artefato encontrado"

U "ksu libs" \
"find /data/adb 2>/dev/null | grep -Ei 'ksu|kernelsu'" \
"ksu" \
"nenhum kernelsu" \
"kernelsu detectado"

U "apatch libs" \
"find /data/adb 2>/dev/null | grep -Ei 'apatch'" \
"apatch" \
"nenhum apatch" \
"apatch detectado"

U "app lib hook" \
"find /data/app 2>/dev/null | grep -Ei 'lib.*(hook|inject|frida)'" \
"hook|inject|frida" \
"apps limpos" \
"lib suspeita dentro de app"

U "system lib hook" \
"find /system /vendor 2>/dev/null | grep -Ei 'lib.*(hook|inject|frida)'" \
"hook|inject|frida" \
"sistema limpo" \
"lib suspeita no sistema"

U "binarios suspeitos" \
"find /data /system 2>/dev/null | grep -Ei 'frida-server|inject|hook'" \
"frida-server|inject|hook" \
"nenhum binário suspeito" \
"binário de injeção encontrado"

U "hidden dot files" \
"find /data 2>/dev/null | grep '/\\.'" \
"/." \
"nenhum oculto relevante" \
"arquivos ocultos encontrados"

U "tmp libs so" \
"find /data/local/tmp 2>/dev/null | grep '\\.so'" \
".so" \
"nenhuma lib tmp" \
"lib .so no tmp detectada"

#         ~ DETECÇÃO  SPOOF         #

U "spoof dmesg file" "find / -type f \\( -iname 'dmesg_spoof' -o -iname '*spoof_dmesg*' -o -iname '*fake_dmesg*' \\) 2>/dev/null" "dmesg_spoof" "Nenhum spoof dmesg" "Arquivo spoof dmesg detectado"

U "spoof cmdline file" "find / -type f \\( -iname 'cmdline_spoof' -o -iname '*spoof_cmdline*' -o -iname '*fake_cmdline*' \\) 2>/dev/null" "cmdline_spoof" "Nenhum spoof cmdline" "Arquivo spoof cmdline detectado"

U "spoof dir check" "find / -type d \\( -iname '*spoof*' -o -iname '*hide*' -o -iname '*rootkit*' -o -iname '*magisk*' \\) 2>/dev/null" "spoof" "Nenhum diretório suspeito" "Diretório spoof/rootkit encontrado"

U "maps spoof check" "cat /proc/*/maps 2>/dev/null | grep -iE 'spoof|fake|hide|rootkit|inject|hook'" "spoof" "Nenhum spoof na memória" "Spoof/hook carregado na memória"

U "cmdline real" "cat /proc/cmdline 2>/dev/null" "androidboot" "Cmdline não acessível" "Cmdline obtido"

U "cmdline security flags" "cat /proc/cmdline 2>/dev/null | grep -Ei 'verifiedbootstate|device_state|veritymode'" "androidboot" "Sem flags de segurança" "Flags de segurança detectadas"

U "boot state check" "getprop ro.boot.verifiedbootstate" "green" "Boot não seguro" "Boot aparentemente seguro"

U "vbmeta check" "getprop ro.boot.vbmeta.device_state" "locked" "Device não locked" "Device locked"

U "verity check" "getprop ro.boot.veritymode" "enforcing" "Verity desativado" "Verity ativo"

U "dmesg real check" "dmesg 2>/dev/null | head -n 20" "Linux version" "dmesg inacessível" "Kernel log obtido"

U "dmesg spoof compare" "dmesg 2>/dev/null | grep -Ei 'SELinux|Linux version'" "Linux" "Sem info kernel" "Kernel info presente"

U "tmp spoof check" "ls /data/local/tmp /data/tmp /sdcard/Download 2>/dev/null | grep -iE 'spoof|fake|mask|su|magisk|busybox|root'" "spoof" "Nenhum spoof no tmp" "Arquivo suspeito no tmp"

U "system spoof file" "find /system /system_ext /product -type f \\( -iname '*spoof*' -o -iname '*hide*' -o -iname '*root*' \\) 2>/dev/null" "spoof" "Nenhum spoof no system" "Spoof dentro do system"

U "vendor spoof file" "find /vendor /odm -type f \\( -iname '*spoof*' -o -iname '*hide*' -o -iname '*root*' \\) 2>/dev/null" "spoof" "Nenhum spoof no vendor" "Spoof dentro do vendor"


U "selinux status" "getenforce 2>/dev/null" "Enforcing" "SELinux permissivo ou desativado" "SELinux em enforcing"

U "rw system mount" "mount | grep ' /system ' | grep -w rw" "rw" "System montado como ro" "System montado como rw (possível modificação)"

U "suspicious kernel modules" "lsmod 2>/dev/null | grep -iE 'rootkit|hide|spoof|inject|hook'" "rootkit" "Nenhum módulo suspeito" "Módulo kernel suspeito carregado"

U "magisk detection" "magisk --path 2>/dev/null || magisk -c 2>/dev/null || ls /data/adb/magisk* 2>/dev/null" "magisk" "Magisk não detectado" "Magisk (root) detectado"

U "su binaries" "find /system /vendor /data /sbin -type f \\( -name su -o -name daemonsu -o -name supolicy \\) 2>/dev/null" "su" "Nenhum binário su encontrado" "Binário su encontrado (root)"

U "xposed detection" "ls /data/data/de.robv.android.xposed.installer 2>/dev/null || ls /data/app/*xposed* 2>/dev/null || ps -A | grep xposed" "xposed" "Xposed não detectado" "Xposed framework presente"

U "busybox location" "busybox --help 2>/dev/null | head -1 | grep -i busybox" "BusyBox" "Busybox não encontrado" "Busybox instalado (possível ferramenta de root)"

U "ld_preload check" "for pid in /proc/[0-9]*; do grep -l 'LD_PRELOAD' \"\$pid/environ\" 2>/dev/null && echo \"\$pid\"; done" "LD_PRELOAD" "Nenhum LD_PRELOAD ativo" "LD_PRELOAD detectado (possível hook)"

U "hidden processes" "ps -A -o pid,args | grep -v grep | grep -E '\\[.*\\]|^\\s*[0-9]+\\s+\\[' || true" "\\[" "Nenhum processo oculto" "Possível processo oculto (thread kernel)"

U "injected libraries" "cat /proc/*/maps 2>/dev/null | grep -iE 'inject|hook|spoof|magisk' | sort -u" "inject" "Nenhuma lib injetada suspeita" "Bibliotecas injetadas detectadas"

U "frida detection" "ps -A | grep -i frida || ls /data/local/tmp/frida* 2>/dev/null || ls /data/app/*frida* 2>/dev/null" "frida" "Frida não detectado" "Frida (ferramenta de hook) detectado"

U "modified system binaries" "for bin in init sh; do if [ -f /system/bin/\$bin ] && [ -f /system/bin/\$bin.bak ] 2>/dev/null; then echo \"\$bin.bak\"; fi; done" "bak" "Nenhum backup suspeito de binário" "Binário do sistema com backup (possível modificação)"

U "kallsyms anomalies" "cat /proc/kallsyms 2>/dev/null | grep -iE 'rootkit|hide|spoof|inject' | head -5" "rootkit" "Símbolos do kernel normais" "Símbolo suspeito no kernel"

U "mount namespace escape" "ls -la /proc/self/ns/mnt 2>/dev/null | grep -v 'self'" "target" "Namespace mount normal" "Possível escape de namespace"

U "process injection via ptrace" "ps -A | grep -E 'ptrace|debuggerd' || true" "ptrace" "Nenhum ptrace suspeito" "Processo com ptrace ativo"

#         ~DETECÇÃO SURF             #

K
Y 35 "SURF DETECTION"

U "zygote maps scan" \
"cat /proc/$(pidof zygote 2>/dev/null)/maps 2>/dev/null | grep -Ei 'zygisk|magisk|ksu|apatch|inject|memfd'" \
"zygisk|magisk|ksu|apatch|inject|memfd" \
"Nenhuma lib suspeita no zygote" \
"Possível injeção SURF no zygote"

U "zygote maps anon" \
"cat /proc/$(pidof zygote 2>/dev/null)/maps 2>/dev/null | grep '00:00 0'" \
"00:00 0" \
"Zygote normal" \
"Memória anônima no zygote (injeção)"

U "mountinfo overlay" \
"cat /proc/self/mountinfo 2>/dev/null | grep -E 'overlay' | grep -E 'upperdir=|workdir=' | grep -E '/system|/vendor|/product'" \
"upperdir=|workdir=" \
"Nenhum overlay suspeito" \
"Overlay oculto detectado"

U "mount hidden tmpfs" \
"cat /proc/self/mountinfo 2>/dev/null | grep -Ei 'tmpfs' | grep -Ei '/system|/vendor|/product'" \
"tmpfs" \
"Nenhum tmpfs suspeito" \
"Montagem tmpfs mascarando partição"

U "mnt namespace global" \
"TOTAL=$(ls /proc | grep -E '^[0-9]+$' | wc -l); NS=$(ls -l /proc/*/ns/mnt 2>/dev/null | awk '{print $NF}' | sort | uniq | wc -l); echo $NS/$TOTAL" \
"^([3-9][0-9]|[1-9][0-9]{2,})/" \
"Namespaces dentro do esperado" \
"Fragmentação alta de namespaces"

U "mnt namespace isolate" \
"SELF=$(readlink /proc/self/ns/mnt); readlink /proc/*/ns/mnt 2>/dev/null | sort | uniq | grep -v \"$SELF\"" \
"." \
"Namespace padrão" \
"Namespaces diferentes do processo (isolamento)"

U "self namespace" \
"readlink /proc/self/ns/mnt" \
"4026531840" \
"Namespace padrão" \
"Namespace isolado detectado"

U "mount propagation" \
"cat /proc/self/mountinfo 2>/dev/null | grep -Ei 'shared|master'" \
"shared" \
"Mount padrão" \
"Mount propagation alterado"

U "suid scan" \
"find /system /vendor /product -perm -4000 2>/dev/null" \
"." \
"Nenhum SUID estranho" \
"Arquivo SUID suspeito detectado"

U "suid data" \
"find /data -perm -4000 2>/dev/null" \
"." \
"Nenhum SUID em /data" \
"SUID fora do padrão em /data"

U "ld_preload" \
"cat /proc/*/environ 2>/dev/null | grep -i LD_PRELOAD" \
"LD_PRELOAD" \
"Nenhum preload detectado" \
"LD_PRELOAD suspeito detectado"

U "ld audit" \
"cat /proc/*/environ 2>/dev/null | grep -i LD_AUDIT" \
"LD_AUDIT" \
"Nenhum LD_AUDIT" \
"LD_AUDIT ativo"

U "root sockets" \
"cat /proc/net/unix 2>/dev/null | grep -Ei 'magisk|ksu|apatch|zygisk'" \
"magisk|ksu|apatch|zygisk" \
"Nenhum socket root" \
"Socket relacionado a root detectado"

U "socket abstract" \
"cat /proc/net/unix 2>/dev/null | grep '@'" \
"@" \
"Nenhum socket abstrato suspeito" \
"Socket abstrato (possível root oculto)"

U "init hooks" \
"grep -Ei 'magisk|ksu|apatch|overlay|mount' /init*.rc 2>/dev/null" \
"magisk|ksu|apatch|aay|mount" \
"Nenhum hook no init.rc" \
"Hook suspeito no init"

U "proc mount scan" \
"cat /proc/self/mounts 2>/dev/null | grep -E 'overlay' | grep -E 'upperdir=|workdir=' | grep -E '/system|/vendor|/product'" \
"upperdir=|workdir=" \
"Nenhum mount suspeito" \
"Mount overlay mascarando partição"

U "bind mount detect" \
"cat /proc/self/mountinfo 2>/dev/null | grep -E 'bind' | grep -E '/system|/vendor|/product'" \
"bind" \
"Sem bind mount suspeito" \
"Bind mount em partição crítica"

U "exec outside system (self)" \
"cat /proc/self/maps 2>/dev/null | grep -E 'r-xp' | grep '00:00 0' | grep -vE 'vdso|vvar|/dev/ashmem|/memfd:jit|/memfd:jit-cache|/memfd:jit-zygote'" \
"r-xp" \
"Execução padrão" \
"Execução anônima no processo (injeção possível)"

U "memfd global" \
"cat /proc/*/maps 2>/dev/null | grep -i memfd" \
"memfd" \
"Nenhum memfd" \
"Execução fileless detectada"

Y 35 "SURF DETECTION"


U "zygote maps scan" "cat /proc/\$(pidof zygote 2>/dev/null)/maps 2>/dev/null | grep -Ei 'zygisk|magisk|ksu|apatch|inject'" "zygisk|magisk|ksu|apatch|inject" "Nenhuma lib suspeita no zygote" "Possível injeção SURF no zygote"


U "mnt namespace" "ls -l /proc/*/ns/mnt 2>/dev/null | grep -v '4026531840'" "mnt" "Namespaces normais" "Namespace mount suspeito"
U "mount namespace isolate" \
"readlink /proc/self/ns/mnt" \
"4026531840" \
"Namespace padrão" \
"Namespace isolado detectado"


U "suid scan" "find /system /vendor /product -perm -4000 2>/dev/null" "." "Nenhum SUID estranho" "Arquivo SUID suspeito detectado"


U "ld_preload" "cat /proc/*/environ 2>/dev/null | grep -i LD_PRELOAD" "LD_PRELOAD" "Nenhum preload detectado" "LD_PRELOAD suspeito detectado"
U "ld audit" \
"cat /proc/self/environ | grep LD_AUDIT" \
"LD_AUDIT" \
"Nenhum LD_AUDIT" \
"LD_AUDIT ativo"


U "root sockets" "cat /proc/net/unix 2>/dev/null | grep -Ei 'magisk|ksu|apatch|zygisk'" "magisk|ksu|apatch|zygisk" "Nenhum socket root" "Socket relacionado a root detectado"


U "init hooks" "grep -Ei 'magisk|ksu|apatch' /init*.rc 2>/dev/null" "magisk|ksu|apatch" "Nenhum hook no init.rc" "Hook root encontrado no init"

U "proc mount scan" "cat /proc/self/mounts | grep -Ei 'tmpfs.*system|tmpfs.*vendor'" "tmpfs" "Nenhum mount suspeito" "Mount mascarado detectado"

#                ~TEE~                #

K
Y 35 "TEE"

U "tee qsee device" \
"ls /dev 2>/dev/null | grep -w qseecom" \
"qseecom" \
"Dispositivo qseecom ausente (suspeito)" \
"TEE QSEE presente"

U "tee optee device" \
"ls /dev 2>/dev/null | grep -w tee" \
"tee" \
"Dispositivo OP-TEE ausente (suspeito)" \
"TEE OP-TEE presente"

U "tee tzdev device" \
"ls /dev 2>/dev/null | grep -w tzdev" \
"tzdev" \
"Driver TrustZone ausente (suspeito)" \
"Driver TrustZone presente"

U "keymaster service" \
"service list 2>/dev/null | grep -w keymaster" \
"keymaster" \
"Keymaster ausente (ANORMAL)" \
"Keymaster ativo"

U "gatekeeper service" \
"service list 2>/dev/null | grep -Ei 'gatekeeper'" \
"^$" \
"Gatekeeper ativo" \
"Gatekeeper ausente (suspeito)"

U "keystore service" \
"service list 2>/dev/null | grep -Ew 'keystore|keystore2'" \
"^$" \
"Keystore ativo" \
"Keystore ausente (suspeito)"

U "tee vendor libs" \
"find /vendor/lib* -type f 2>/dev/null | grep -Ei 'tee|qsee|trusty|keymaster'" \
"^$" \
"Libs TEE presentes" \
"Libs TEE ausentes (possível bypass)"

U "tee firmware check" \
"ls /vendor/firmware 2>/dev/null | grep -Ei 'tz|tee|qsee'" \
"tz|tee|qsee" \
"Firmware TEE ausente (suspeito)" \
"Firmware TEE presente"

U "tee kernel logs" \
"dmesg 2>/dev/null | grep -Ei 'qsee|trustzone|optee|tee'" \
"qsee|trustzone|optee|tee" \
"Sem logs TEE (pode ser bloqueado)" \
"TEE registrado no kernel"

U "tee proc driver" \
"ls /proc 2>/dev/null | grep -Ew 'tz|qsee|tee|trusty'" \
"^$" \
"Driver TEE presente" \
"Driver TEE ausente (possível bypass)"

U "tee props check" \
"getprop 2>/dev/null | grep -Ei 'ro.hardware.keymaster|ro.hardware.keystore|keymaster|gatekeeper'" \
"^$" \
"Props TEE presentes" \
"Sem props críticas TEE"

U "tee mismatch" \
"sh -c 'ls /dev | grep -Ei \"qsee|tee\" && service list | grep keymaster'" \
"keymaster" \
"Inconsistência TEE detectada" \
"Stack TEE consistente"

U "tee missing critical" \
"service list 2>/dev/null | grep -Ew 'keymaster|gatekeeper'" \
"^$" \
"Serviços críticos OK" \
"Serviços críticos ausentes (FORTE SUSPEITA)"

#               ~BOOTLOADER~                #

K
Y 35 "BOOTLOADER"

U "bootloader unlocked prop" \
"getprop ro.boot.flash.locked 2>/dev/null" \
"0" \
"Bootloader bloqueado" \
"Bootloader desbloqueado"

U "bootloader verifiedboot" \
"getprop ro.boot.verifiedbootstate 2>/dev/null" \
"orange" \
"Verified Boot OK" \
"Verified Boot comprometido"

U "bootloader vbmeta" \
"getprop ro.boot.vbmeta.device_state 2>/dev/null" \
"unlocked" \
"VBMeta normal" \
"VBMeta desbloqueado"

U "bootloader warranty" \
"getprop ro.boot.warranty_bit 2>/dev/null" \
"1" \
"Warranty intacta" \
"Warranty bit alterado"

U "bootloader secure" \
"getprop ro.secure 2>/dev/null" \
"0" \
"Sistema seguro" \
"Sistema não seguro (ro.secure=0)"

U "bootloader debug" \
"getprop ro.debuggable 2>/dev/null" \
"1" \
"Sistema não debug" \
"Sistema debug ativo"

U "bootloader adb root" \
"getprop ro.adb.secure 2>/dev/null" \
"0" \
"ADB seguro" \
"ADB root permitido"

U "cmdline unlock" \
"cat /proc/cmdline 2>/dev/null | grep -Ei 'androidboot.verifiedbootstate=orange|androidboot.flash.locked=0'" \
"orange|flash.locked=0" \
"Boot normal" \
"Flag de bootloader desbloqueado detectada"

U "fastboot traces" \
"getprop 2>/dev/null | grep -Ei 'fastboot'" \
"fastboot" \
"Sem traços fastboot" \
"Referência a fastboot detectada"

U "avb status" \
"getprop ro.boot.avb_version 2>/dev/null" \
"avb" \
"AVB presente" \
"AVB ausente ou alterado"

U "boot state check" \
"getprop ro.boot.bootstate 2>/dev/null" \
"orange" \
"Boot state normal" \
"Boot state comprometido"

U "device state combo" \
"sh -c 'getprop ro.boot.flash.locked; getprop ro.boot.verifiedbootstate'" \
"0.*orange" \
"Estado consistente" \
"Inconsistência bootloader detectada"

#                   ~RIRU~                    #

K
Y 35 "RIRU"

U "riru pkg" \
"pm list packages 2>/dev/null | grep -i riru" \
"riru" \
"Nenhum Riru" \
"Riru instalado"

U "riru modules" \
"ls /data/adb/modules 2>/dev/null | grep -i riru" \
"riru" \
"Nenhum módulo Riru" \
"Riru nos módulos"

U "riru core dir" \
"ls /data/adb/modules/riru* 2>/dev/null" \
"riru" \
"Diretório Riru não encontrado" \
"Diretório Riru detectado"

U "riru lib files" \
"find /data /system /vendor 2>/dev/null | grep -Ei 'libriru|riru.*\\.so'" \
"riru" \
"Nenhuma lib Riru" \
"Biblioteca Riru detectada"

U "riru tmp" \
"ls /data/local/tmp 2>/dev/null | grep -i riru" \
"riru" \
"Nenhum Riru no tmp" \
"Riru no tmp detectado"

U "riru cache" \
"find /data/data 2>/dev/null | grep -Ei 'riru'" \
"riru" \
"Nenhum cache Riru" \
"Riru encontrado em dados de app"

U "riru maps self" \
"cat /proc/self/maps 2>/dev/null | grep -i riru" \
"riru" \
"Nenhum Riru na memória" \
"Riru carregado na memória"

U "riru maps global" \
"cat /proc/*/maps 2>/dev/null | grep -i riru" \
"riru" \
"Nenhum Riru global" \
"Riru detectado em memória global"

U "riru zygote hook" \
"cat /proc/*/maps 2>/dev/null | grep -Ei 'zygote|app_process' | grep -i riru" \
"riru" \
"Nenhum hook Riru" \
"Riru injetado no zygote"

U "riru props" \
"getprop 2>/dev/null | grep -i riru" \
"riru" \
"Nenhuma prop Riru" \
"Propriedade Riru detectada"

U "riru bin scan" \
"find /data /system 2>/dev/null | grep -Ei 'riru'" \
"riru" \
"Nenhum binário Riru" \
"Arquivo Riru detectado"

U "riru hide check" \
"ls /data/adb/modules 2>/dev/null | grep -Ei 'riru.*hide'" \
"riru" \
"Nenhum hide Riru" \
"Riru com ocultação detectado"

#                 ~RAMDISK~                 #

U "ramdisk cpio scan" 
"find /dev/block -name "boot*" -exec cat {} + 2>/dev/null | grep -a "070701"" 
"070701" 
"assinatura CPIO (Ramdisk) detectada via varredura" 
"ramdisk padrão ou protegido"
U "magisk modules config" 
"find /data/adb/modules -name "module.prop" -exec cat {} + 2>/dev/null | grep -a "id="" 
"id=" 
"módulos magisk ativos identificados por conteúdo" 
"nenhum módulo magisk configurado"
U "magisk binaries fingerprint" 
"find /data/local/tmp /data/adb -type f -exec cat {} + 2>/dev/null | grep -a "MAGISK_VER_CODE"" 
"MAGISK_VER_CODE" 
"binário magisk identificado por assinatura interna" 
"nenhum binário magisk detectado"
U "ksu kernel symbols" 
"cat /proc/kallsyms 2>/dev/null | grep -a "ksu_"" 
"ksu_" 
"símbolos do KernelSU ativos na memória" 
"kernel limpo ou kallsyms restrito"
U "init binary patch" 
"cat /init 2>/dev/null | grep -a "magisk"" 
"magisk" 
"binário init patcheado (vulnerabilidade alta)" 
"binário init original"
U "lkm root modules" 
"find /lib/modules /data/adb -name "*.ko" -exec cat {} + 2>/dev/null | grep -a "vermagic"" 
"vermagic" 
"módulos de kernel (LKM) de root detectados" 
"nenhum módulo LKM suspeito encontrado"

#                 ~TRICKSTORE~                 #

U "trickstore config detect" 
"ls /data/adb/tricky_store/target.txt /data/adb/tricky_store/keybox.xml 2>/dev/null" 
"tricky_store" 
"arquivos de configuração do TrickStore encontrados" 
"nenhum rastro de configuração do TrickStore"
U "trickstore module check" 
"cat /data/adb/modules/tricky_store/module.prop 2>/dev/null | grep -i "TrickStore"" 
"TrickStore" 
"módulo TrickStore ativo detectado" 
"módulo TrickStore não instalado"
U "keymint spoofing rastro" 
"find /data/adb/tricky_store -type f -exec cat {} + 2>/dev/null | grep -a "EC_PRIVATE_KEY"" 
"EC_PRIVATE_KEY" 
"chave privada de Keybox (spoofing) encontrada no TrickStore" 
"nenhuma chave de spoofing de Keybox detectada"
U "trickstore binary hunt" 
"find /data/adb -name "tricky" 2>/dev/null" 
"tricky" 
"binário ou biblioteca do TrickStore identificada" 
"nenhuma biblioteca 'tricky' encontrada"
U "prop spoofing check" 
"getprop | grep -E "fingerprint|model|product" | grep -i "pixel"" 
"pixel" 
"spoofing de modelo (possível TrickStore/Pif) ativo" 
"propriedades de hardware parecem originais"

#                 ~KEYBOX~                 #

U "keybox file search" 
"find /data/adb /data/local/tmp -name "keybox.xml" 2>/dev/null" 
"keybox.xml" 
"arquivo de certificação Keybox encontrado" 
"nenhuma Keybox física detectada"
U "keybox content scan" 
"find /data/adb -type f -exec cat {} + 2>/dev/null | grep -a "<Keybox"" 
"<Keybox" 
"assinatura XML de Keybox detectada no conteúdo" 
"nenhum cabeçalho de Keybox identificado"
U "keybox certificate chain" 
"find /data/adb -type f -exec cat {} + 2>/dev/null | grep -a "BEGIN CERTIFICATE"" 
"BEGIN CERTIFICATE" 
"cadeia de certificados (spoofing) encontrada em arquivos de root" 
"nenhum certificado de terceiro encontrado em diretórios de root"
U "pif keybox rastro" 
"cat /data/adb/modules/playintegrityfix/pif.json 2>/dev/null | grep -E "model|fingerprint"" 
"model" 
"configuração de spoofing de Keybox (PIF) ativa" 
"pif.json não encontrado ou sem spoofing ativo"
U "keybox private key" 
"find /data/adb -type f -exec cat {} + 2>/dev/null | grep -a "EC_PRIVATE_KEY"" 
"EC_PRIVATE_KEY" 
"chave privada de atestação (Keybox) encontrada" 
"nenhuma chave privada de spoofing detectada"

U "keybox file search" 
"find /data/adb /data/local/tmp -name "keybox.xml" 2>/dev/null" 
"keybox.xml" 
"arquivo de certificação Keybox encontrado" 
"nenhuma Keybox física detectada"
U "keybox content scan" 
"find /data/adb -type f -size -50k -exec grep -l "<Keybox" {} + 2>/dev/null" 
"keybox" 
"assinatura XML de Keybox detectada no conteúdo" 
"nenhum cabeçalho de Keybox identificado"
U "keybox certificate chain" 
"find /data/adb -type f -exec grep -l "BEGIN CERTIFICATE" {} + 2>/dev/null" 
"data/adb" 
"cadeia de certificados (spoofing) encontrada em arquivos de root" 
"nenhum certificado de terceiro encontrado em diretórios de root"
U "pif/spoofing config scan" 
"find /data/adb/modules -name ".json" -o -name ".xml" 2>/dev/null | xargs grep -E "model|fingerprint|product"" 
"model" 
"configuração de spoofing ativa (PIF/TrickStore)" 
"nenhuma config de spoofing de hardware detectada"
U "keybox private key" 
"find /data/adb -type f -exec grep -l "EC_PRIVATE_KEY" {} + 2>/dev/null" 
"EC_PRIVATE_KEY" 
"chave privada de atestação (Keybox) encontrada" 
"nenhuma chave privada de spoofing detectada"
echo ""
echo -e "\033[1;36m╔══════════════════════════════════╗"
echo -e "║   ██████╗  ██████╗  ██████╗     ║"
echo -e "║   ██╔══██╗██╔════╝ ██╔════╝     ║"
echo -e "║   ██║  ██║██║  ███╗╚█████╗      ║"
echo -e "║   ██║  ██║██║   ██║ ╚═══██╗     ║"
echo -e "║   ██████╔╝╚██████╔╝██████╔╝     ║"
echo -e "║   ╚═════╝  ╚═════╝ ╚═════╝      ║"
echo -e "║                                ║"
echo -e "║        🔹 DG7 SS FINALIZADO 🔹     ║"
echo -e "╚══════════════════════════════════╝\033[0m"
