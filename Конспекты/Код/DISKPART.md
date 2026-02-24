``` Python
class DiskpartSimulator:

    """Расширенный эмулятор diskpart для тренировки команд"""

    def __init__(self):

        self.current_disk = None

        self.current_partition = None

        self.current_volume = None

        self.disks = {}

        self.partitions = {}

        self.volumes = {}

        self.init_sample_data()

        self.command_history = []

    def init_sample_data(self):

        """Инициализация тестовых данных"""

        # Диски

        self.disks = {

            0: {

                "size": "250 GB",

                "free": "250 GB",

                "status": "Online",

                "gpt": False,

                "dynamic": False,

                "style": "MBR"

            },

            1: {

                "size": "1000 GB",

                "free": "1000 GB",

                "status": "Online",

                "gpt": True,

                "dynamic": False,

                "style": "GPT"

            },

            2: {

                "size": "32 GB",

                "free": "32 GB",

                "status": "Offline",

                "gpt": False,

                "dynamic": False,

                "style": "MBR"

            },

            3: {

                "size": "500 GB",

                "free": "200 GB",

                "status": "Online",

                "gpt": False,

                "dynamic": True,

                "style": "Dynamic"

            }

        }

        # Разделы для диска 3 (пример с существующими разделами)

        self.partitions[3] = [

            {

                "number": 1,

                "size": "100 MB",

                "type": "System",

                "offset": "1024 KB",

                "active": True

            },

            {

                "number": 2,

                "size": "200 GB",

                "type": "Primary",

                "offset": "101 MB",

                "active": False

            }

        ]

        # Тома (логические диски)

        self.volumes = {

            "C": {

                "type": "Simple",

                "size": "200 GB",

                "status": "Healthy",

                "info": "System",

                "disk": 3,

                "partition": 2,

                "fs": "NTFS",

                "label": "Windows"

            },

            "D": {

                "type": "Simple",

                "size": "500 GB",

                "status": "Healthy",

                "info": "Data",

                "disk": 1,

                "partition": 1,

                "fs": "NTFS",

                "label": "Storage"

            },

            "E": {

                "type": "CD-ROM",

                "size": "700 MB",

                "status": "Healthy",

                "info": "DVD",

                "disk": None,

                "partition": None,

                "fs": "UDF",

                "label": "Windows Setup"

            }

        }

    def show_help(self):

        """Показывает справку по всем командам"""

        help_text = """

╔══════════════════════════════════════════════════════════════╗

║                   DISKPART СИМУЛЯТОР - HELP                  ║

╠══════════════════════════════════════════════════════════════╣

║                     ║  КОМАНДЫ УПРАВЛЕНИЯ ДИСКАМИ            ║

╠═════════════════════╬════════════════════════════════════════╣

║ list disk           ║  Показать все диски                    ║

║ select disk N       ║  Выбрать диск N                        ║

║ detail disk         ║  Детальная информация о диске          ║

║ clean               ║  Очистить диск (удалить все разделы)   ║

║ convert mbr/gpt     ║  Конвертировать стиль разделов         ║

║ online disk         ║  Включить диск онлайн                  ║

║ offline disk        ║  Отключить диск                        ║

║ attributes disk     ║  Показать атрибуты диска               ║

╠═════════════════════╬════════════════════════════════════════╣

║                     ║  КОМАНДЫ УПРАВЛЕНИЯ РАЗДЕЛАМИ          ║

╠═════════════════════╬════════════════════════════════════════╣

║ list partition      ║  Показать разделы выбранного диска     ║

║ select partition N  ║  Выбрать раздел N                      ║

║ detail partition    ║  Детали раздела                        ║

║ create partition    ║  Создать раздел:                       ║

║   primary size=N    ║    Основной раздел размером N (MB)     ║

║   extended          ║    Расширенный раздел                  ║

║   logical size=N    ║    Логический диск в расширенном       ║

║   efi size=100      ║    Системный раздел EFI (GPT)          ║

║   msr size=128      ║    Зарезервированный раздел (GPT)      ║

║ delete partition    ║  Удалить выбранный раздел              ║

║ active              ║  Сделать раздел активным (для загрузки)║

║ inactive            ║  Убрать активность                     ║

╠═════════════════════╬════════════════════════════════════════╣

║                     ║  КОМАНДЫ УПРАВЛЕНИЯ ТОМАМИ             ║

╠═════════════════════╬════════════════════════════════════════╣

║ list volume         ║  Показать все тома                     ║

║ select volume N/X   ║  Выбрать том (по номеру или букве)     ║

║ detail volume       ║  Детали тома                           ║

║ assign letter=X     ║  Назначить букву диска                 ║

║ remove letter=X     ║  Удалить букву диска                   ║

║ format              ║  Форматирование:                       ║

║   fs=ntfs quick     ║    Быстрое форматирование в NTFS       ║

║   fs=fat32 label=X  ║    FAT32 с меткой                      ║

║ extend              ║  Расширить том за счет свободного места║

║ shrink desired=N    ║  Уменьшить том на N (MB)               ║

║ delete volume       ║  Удалить том                           ║

╠═════════════════════╬════════════════════════════════════════╣

║                     ║  СЛУЖЕБНЫЕ КОМАНДЫ                     ║

╠═════════════════════╬════════════════════════════════════════╣

║ exit                ║  Выход из симулятора                   ║

║ history             ║  Показать историю команд               ║

║ cls                 ║  Очистить экран                        ║

║ help/?              ║  Показать эту справку                  ║

╚══════════════════════════════════════════════════════════════╝

        """

        print(help_text)

    def list_disks(self):

        """Команда list disk"""

        print("\n  Диск ###  Состояние  Размер   Свободно  Дин  GPT")

        print("  ---------  ---------  -------  --------  ---  ---")

        for num, info in self.disks.items():

            dyn_mark = "Да" if info["dynamic"] else "Нет"

            gpt_mark = "*" if info["gpt"] else " "

            print(f"  Диск {num}    {info['status']}   {info['size']:>7}  {info['free']:>8}  {dyn_mark:<3}  {gpt_mark}")

    def select_disk(self, args):

        """Команда select disk N"""

        try:

            disk_num = int(args.split()[-1])

            if disk_num in self.disks:

                self.current_disk = disk_num

                self.current_partition = None

                print(f"\n  Диск {disk_num} выбран.")

                print(f"  Диск {disk_num} - {self.disks[disk_num]['style']}, {self.disks[disk_num]['size']}")

            else:

                print(f"\n  Указанный диск {disk_num} не найден.")

        except:

            print("\n  Ошибка синтаксиса. Используйте: select disk N")

    def detail_disk(self):

        """Команда detail disk"""

        if self.current_disk is None:

            print("\n  Диск не выбран. Сначала используйте select disk.")

            return

        disk = self.disks[self.current_disk]

        print(f"\n  {disk['style']} диск {self.current_disk}")

        print(f"  Идентификатор диска: {hex(hash(str(disk)))}")

        print(f"  Тип: {disk['style']}")

        print(f"  Состояние: {disk['status']}")

        print(f"  Размер: {disk['size']}")

        print(f"  Свободно: {disk['free']}")

        print(f"  Используемый стиль разделов: {'GPT' if disk['gpt'] else 'MBR'}")

        print(f"  Динамический: {'Да' if disk['dynamic'] else 'Нет'}")

        # Показываем разделы

        self.list_partitions()

    def list_partitions(self):

        """Команда list partition"""

        if self.current_disk is None:

            print("\n  Диск не выбран. Сначала используйте select disk.")

            return

        print(f"\n  Раздел ###  Тип          Размер    Смещение")

        print(f"  ----------  ------------  --------  --------")

        partitions = self.partitions.get(self.current_disk, [])

        if not partitions:

            print("  В этом диске нет разделов.")

        else:

            for part in partitions:

                active_mark = "Активный" if part.get("active", False) else ""

                print(f"  Раздел {part['number']}    {part['type']:<12} {part['size']:>8}  {part['offset']:>8}  {active_mark}")

    def select_partition(self, args):

        """Команда select partition N"""

        try:

            part_num = int(args.split()[-1])

            if self.current_disk is None:

                print("\n  Сначала выберите диск.")

                return

            partitions = self.partitions.get(self.current_disk, [])

            for part in partitions:

                if part["number"] == part_num:

                    self.current_partition = part_num

                    print(f"\n  Раздел {part_num} выбран.")

                    print(f"  Тип: {part['type']}, Размер: {part['size']}")

                    return

            print(f"\n  Раздел {part_num} не найден на диске {self.current_disk}.")

        except:

            print("\n  Ошибка синтаксиса. Используйте: select partition N")

    def create_partition(self, args):

        """Команда create partition primary/extended/logical/efi/msr size=N"""

        if self.current_disk is None:

            print("\n  Сначала выберите диск.")

            return

        args_lower = args.lower()

        # Парсим размер

        size_match = __import__('re').search(r'size[=\s](\d+)', args_lower)

        size = int(size_match.group(1)) if size_match else None

        disk = self.disks[self.current_disk]

        if "primary" in args_lower:

            part_type = "Primary"

        elif "extended" in args_lower:

            part_type = "Extended"

        elif "logical" in args_lower:

            part_type = "Logical"

        elif "efi" in args_lower:

            part_type = "EFI System"

        elif "msr" in args_lower:

            part_type = "MSR (Reserved)"

        else:

            print("\n  Укажите тип раздела: primary, extended, logical, efi, msr")

            return

        # Проверка стиля раздела

        if part_type in ["EFI System", "MSR (Reserved)"] and not disk["gpt"]:

            print(f"\n  Ошибка: {part_type} доступен только на GPT дисках.")

            return

        # Создаем новый раздел

        new_part_num = len(self.partitions.get(self.current_disk, [])) + 1

        if self.current_disk not in self.partitions:

            self.partitions[self.current_disk] = []

        size_str = f"{size} MB" if size else "100% диска"

        new_part = {

            "number": new_part_num,

            "size": size_str,

            "type": part_type,

            "offset": "1024 KB",

            "active": False

        }

        self.partitions[self.current_disk].append(new_part)

        print(f"\n  Раздел {new_part_num} создан успешно.")

        print(f"  Тип: {part_type}, Размер: {size_str}")

    def list_volumes(self):

        """Команда list volume"""

        print("\n  Том ###  Буква  Метка        ФС     Тип       Размер   Состояние")

        print("  -------  -----  -----------  -----  --------  -------  ---------")

        vol_num = 1

        for letter, info in self.volumes.items():

            print(f"  Том {vol_num:<5} {letter:<6} {info['label']:<11} {info['fs']:<6} {info['type']:<8} {info['size']:<7} {info['status']}")

            vol_num += 1

    def select_volume(self, args):

        """Команда select volume N/X"""

        try:

            selector = args.split()[-1]

            # Поиск по букве

            if len(selector) == 1 and selector.isalpha():

                letter = selector.upper()

                if letter in self.volumes:

                    self.current_volume = letter

                    print(f"\n  Том {letter} выбран.")

                    vol = self.volumes[letter]

                    print(f"  {vol['label']}, {vol['size']}, {vol['fs']}")

                    return

            # Поиск по номеру

            vol_num = int(selector)

            vol_list = list(self.volumes.keys())

            if 1 <= vol_num <= len(vol_list):

                self.current_volume = vol_list[vol_num-1]

                print(f"\n  Том {vol_num} выбран.")

                vol = self.volumes[self.current_volume]

                print(f"  Буква: {self.current_volume}, {vol['label']}, {vol['size']}")

                return

            print(f"\n  Том {selector} не найден.")

        except:

            print("\n  Ошибка синтаксиса. Используйте: select volume N или select volume X")

    def assign_letter(self, args):

        """Команда assign letter=X"""

        if self.current_volume is None:

            print("\n  Сначала выберите том.")

            return

        match = __import__('re').search(r'letter[=\s](\w)', args)

        if not match:

            print("\n  Укажите букву: assign letter=X")

            return

        new_letter = match.group(1).upper()

        old_letter = self.current_volume

        if new_letter in self.volumes and new_letter != old_letter:

            print(f"\n  Буква {new_letter} уже используется.")

            return

        # Переименовываем том

        if new_letter != old_letter:

            self.volumes[new_letter] = self.volumes.pop(old_letter)

            self.current_volume = new_letter

            print(f"\n  Тому {old_letter} назначена буква {new_letter}.")

    def format_volume(self, args):

        """Команда format fs=ntfs/fat32 quick label=name"""

        if self.current_volume is None:

            print("\n  Сначала выберите том.")

            return

        args_lower = args.lower()

        # Парсим файловую систему

        fs_match = __import__('re').search(r'fs[=\s](\w+)', args_lower)

        fs = fs_match.group(1) if fs_match else "ntfs"

        # Парсим метку

        label_match = __import__('re').search(r'label[=\s][\'"]?([^\'"]+)[\'"]?', args)

        label = label_match.group(1) if label_match else ""

        # Проверка на quick

        quick = "quick" in args_lower

        vol = self.volumes[self.current_volume]

        old_fs = vol["fs"]

        old_label = vol["label"]

        vol["fs"] = fs.upper()

        if label:

            vol["label"] = label

        print(f"\n  Форматирование тома {self.current_volume}...")

        print(f"  Файловая система: {fs.upper()} (было {old_fs})")

        if label:

            print(f"  Метка тома: {label} (было {old_label})")

        print(f"  Режим: {'Быстрое' if quick else 'Полное'} форматирование")

        print(f"  Форматирование завершено.")

    def delete_partition(self):

        """Команда delete partition"""

        if self.current_disk is None:

            print("\n  Сначала выберите диск.")

            return

        if self.current_partition is None:

            print("\n  Сначала выберите раздел.")

            return

        confirm = input(f"  Удалить раздел {self.current_partition} на диске {self.current_disk}? (yes/no): ")

        if confirm.lower() == "yes":

            partitions = self.partitions.get(self.current_disk, [])

            self.partitions[self.current_disk] = [p for p in partitions if p["number"] != self.current_partition]

            print(f"\n  Раздел {self.current_partition} удален.")

            self.current_partition = None

        else:

            print("\n  Операция отменена.")

    def clean_disk(self):

        """Команда clean"""

        if self.current_disk is None:

            print("\n  Сначала выберите диск.")

            return

        confirm = input(f"\n  ⚠️  ВНИМАНИЕ: Все данные на диске {self.current_disk} будут удалены!\n  Продолжить? (yes/no): ")

        if confirm.lower() == "yes":

            self.partitions[self.current_disk] = []

            self.disks[self.current_disk]["free"] = self.disks[self.current_disk]["size"]

            print(f"\n  Диск {self.current_disk} очищен успешно.")

        else:

            print("\n  Операция отменена.")

    def convert_disk(self, args):

        """Команда convert mbr/gpt"""

        if self.current_disk is None:

            print("\n  Сначала выберите диск.")

            return

        args_lower = args.lower()

        disk = self.disks[self.current_disk]

        if "gpt" in args_lower:

            if disk["gpt"]:

                print(f"\n  Диск {self.current_disk} уже в формате GPT.")

            else:

                disk["gpt"] = True

                disk["style"] = "GPT"

                print(f"\n  Диск {self.current_disk} конвертирован в GPT.")

        elif "mbr" in args_lower:

            if not disk["gpt"]:

                print(f"\n  Диск {self.current_disk} уже в формате MBR.")

            else:

                disk["gpt"] = False

                disk["style"] = "MBR"

                print(f"\n  Диск {self.current_disk} конвертирован в MBR.")

        else:

            print("\n  Укажите формат: convert mbr или convert gpt")

    def show_history(self):

        """Показывает историю команд"""

        if not self.command_history:

            print("\n  История команд пуста.")

            return

        print("\n  История команд:")

        for i, cmd in enumerate(self.command_history[-10:], 1):

            print(f"  {i:2d}. {cmd}")

    def clear_screen(self):

        """Очистка экрана"""

        __import__('os').system('cls' if __import__('os').name == 'nt' else 'clear')

        self.show_help()

    def run(self):

        """Главный цикл обработки команд"""

        self.show_help()

        print("\n  Diskpart Simulator запущен. Введите 'help' для справки.")

        while True:

            try:

                cmd = input("\nDISKPART> ").strip()

                if not cmd:

                    continue

                self.command_history.append(cmd)

                cmd_lower = cmd.lower()

                # Обработка команд

                if cmd_lower in ["exit", "quit"]:

                    print("\n  Выход из симулятора.")

                    break

                elif cmd_lower == "help" or cmd_lower == "?":

                    self.show_help()

                elif cmd_lower == "list disk":

                    self.list_disks()

                elif cmd_lower.startswith("select disk"):

                    self.select_disk(cmd)

                elif cmd_lower == "detail disk":

                    self.detail_disk()

                elif cmd_lower == "list partition":

                    self.list_partitions()

                elif cmd_lower.startswith("select partition"):

                    self.select_partition(cmd)

                elif cmd_lower == "detail partition":

                    if self.current_partition:

                        for part in self.partitions.get(self.current_disk, []):

                            if part["number"] == self.current_partition:

                                print(f"\n  Раздел {self.current_partition}")

                                print(f"  Тип: {part['type']}")

                                print(f"  Размер: {part['size']}")

                                print(f"  Смещение: {part['offset']}")

                                print(f"  Активный: {'Да' if part.get('active', False) else 'Нет'}")

                                break

                    else:

                        print("\n  Раздел не выбран.")

                elif cmd_lower.startswith("create partition"):

                    self.create_partition(cmd)

                elif cmd_lower == "delete partition":

                    self.delete_partition()

                elif cmd_lower == "active":

                    if self.current_partition:

                        for part in self.partitions.get(self.current_disk, []):

                            if part["number"] == self.current_partition:

                                part["active"] = True

                                print(f"\n  Раздел {self.current_partition} помечен как активный.")

                                break

                    else:

                        print("\n  Сначала выберите раздел.")

                elif cmd_lower == "inactive":

                    if self.current_partition:

                        for part in self.partitions.get(self.current_disk, []):

                            if part["number"] == self.current_partition:

                                part["active"] = False

                                print(f"\n  Активность раздела {self.current_partition} снята.")

                                break

                    else:

                        print("\n  Сначала выберите раздел.")

                elif cmd_lower == "list volume":

                    self.list_volumes()

                elif cmd_lower.startswith("select volume"):

                    self.select_volume(cmd)

                elif cmd_lower == "detail volume":

                    if self.current_volume:

                        vol = self.volumes[self.current_volume]

                        print(f"\n  Том {self.current_volume}")

                        print(f"  Буква: {self.current_volume}")

                        print(f"  Метка: {vol['label']}")

                        print(f"  Файловая система: {vol['fs']}")

                        print(f"  Размер: {vol['size']}")

                        print(f"  Состояние: {vol['status']}")

                        print(f"  Тип: {vol['type']}")

                        if vol['disk']:

                            print(f"  Диск: {vol['disk']}, Раздел: {vol['partition']}")

                    else:

                        print("\n  Том не выбран.")

                elif cmd_lower.startswith("assign"):

                    self.assign_letter(cmd)

                elif cmd_lower.startswith("remove"):

                    if self.current_volume:

                        letter = self.current_volume

                        # Создаем новый том с другим именем

                        self.volumes[f"Том{len(self.volumes)+1}"] = self.volumes.pop(letter)

                        print(f"\n  Буква {letter} удалена.")

                        self.current_volume = None

                    else:

                        print("\n  Сначала выберите том.")

                elif cmd_lower.startswith("format"):

                    self.format_volume(cmd)

                elif cmd_lower == "clean":

                    self.clean_disk()

                elif cmd_lower.startswith("convert"):

                    self.convert_disk(cmd)

                elif cmd_lower == "online disk":

                    if self.current_disk:

                        self.disks[self.current_disk]["status"] = "Online"

                        print(f"\n  Диск {self.current_disk} включен.")

                    else:

                        print("\n  Сначала выберите диск.")

                elif cmd_lower == "offline disk":

                    if self.current_disk:

                        self.disks[self.current_disk]["status"] = "Offline"

                        print(f"\n  Диск {self.current_disk} отключен.")

                    else:

                        print("\n  Сначала выберите диск.")

                elif cmd_lower == "history":

                    self.show_history()

                elif cmd_lower == "cls":

                    self.clear_screen()

                elif cmd_lower.startswith("extend"):

                    if self.current_volume:

                        print(f"\n  Том {self.current_volume} расширен на всё свободное место.")

                    else:

                        print("\n  Сначала выберите том.")

                elif cmd_lower.startswith("shrink"):

                    match = __import__('re').search(r'desired[=\s](\d+)', cmd_lower)

                    if match and self.current_volume:

                        size = match.group(1)

                        print(f"\n  Том {self.current_volume} уменьшен на {size} MB.")

                    else:

                        print("\n  Используйте: shrink desired=N")

                elif cmd_lower == "delete volume":

                    if self.current_volume:

                        confirm = input(f"  Удалить том {self.current_volume}? (yes/no): ")

                        if confirm.lower() == "yes":

                            del self.volumes[self.current_volume]

                            print(f"\n  Том {self.current_volume} удален.")

                            self.current_volume = None

                    else:

                        print("\n  Сначала выберите том.")

                elif cmd_lower == "attributes disk":

                    if self.current_disk:

                        disk = self.disks[self.current_disk]

                        print(f"\n  Атрибуты диска {self.current_disk}:")

                        print(f"  Только для чтения: Нет")

                        print(f"  Скрытый: Нет")

                        print(f"  Системный: Нет")

                    else:

                        print("\n  Сначала выберите диск.")

                else:

                    print(f"\n  Команда '{cmd}' не распознана. Введите 'help' для справки.")

            except KeyboardInterrupt:

                print("\n\n  Выход по Ctrl+C.")

                break

            except Exception as e:

                print(f"\n  Ошибка: {e}")

  

if __name__ == "__main__":

    simulator = DiskpartSimulator()

    simulator.run()
   ```