# Содержание

1. [Описание](#Описание)
2. [Подготовка проекта](#подготовка-проекта)
3. [Создание тура](#создание-тура)
4. [Загрузка на платформу Tourverse](#Загрузка-на-платформу-Tourverse)
5. [Возможные проблемы и решения](#Возможные-проблемы-и-решения)
6. [Часто задаваемые вопросы](#Часто-задаваемые-вопросы)
7. [Приложение](#Приложение)

# Описание
### 1. Tourverse платформа





### 2. Туры





### 3. Цель документации






# Подготовка проекта
### 1. Требования к проекту
- [Unity 6000.2.6f2](https://unity.com/ru/releases/editor/whats-new/6000.2.6f2)
- [ini-dev2/TourverseToolkitDoc](https://github.com/ini-dev2/TourverseToolkitDoc)
- [OpenXR Plugin | OpenXR Plugin | 1.15.1](https://docs.unity3d.com/Packages/com.unity.xr.openxr@1.15/manual/index.html)
- https://github.com/focus-creative-games/hybridclr_unity.git#v8.7.0
- [Addressables package | Addressables | 2.7.6](https://docs.unity3d.com/Packages/com.unity.addressables@2.7/manual/index.html)
- Название тура, переданное от платформы
### 2. Настройка среды

1. Установка необходимых зависимостей
- [ini-dev2/TourverseToolkitDoc](https://github.com/ini-dev2/TourverseToolkitDoc)
- [OpenXR Plugin | OpenXR Plugin | 1.15.1](https://docs.unity3d.com/Packages/com.unity.xr.openxr@1.15/manual/index.html)
- https://github.com/focus-creative-games/hybridclr_unity.git#v8.7.0
- [Addressables package | Addressables | 2.7.6](https://docs.unity3d.com/Packages/com.unity.addressables@2.7/manual/index.html)

2. Меняем платформу проекта на Android

   ![](Res/Pasted%20image%2020260115150051.png)<br>
   ![](Res/Pasted%20image%2020260115150207.png)<br>
   
3. Настраиваем конфигурацию API Compatibility Level на .NET Framework

   ![](Res/Pasted%20image%2020260115145900.png)<br>
   
5. Если не загружен пакет HybridCLR, необходимо загрузить.  После установки в поле Installed Version должно быть текущая установленная

   ![](Res/Pasted%20image%2020260115150331.png)<br>
   ![](Res/Pasted%20image%2020260115150357.png)
   
7. Для работы hot-кода в папке Scripts нужно создать Assembly Definition с названием, выданным от платформы. И добавить его в настройки HybridCLR

   ![](Res/Pasted%20image%2020260115150904.png)<br>
   ![](Res/Pasted%20image%2020260115151212.png)
   
9. Для сборки проекта в настройках TourverseToolkit назначаем поля
   Tour Name - название тура, переданное от платформы (Обязательный)
   Tour Scene - asset игровой сцены (Обязательный)
   Tour Assembly - asset Assembly Definition скриптов (Обязательный)
   Tour Addressables Uuid - ссылка на удаленные ресурсы тура (Опциональный), если его нет назначается базовый {AddressablesUuid}
   
   ![](Res/Pasted%20image%2020260115151424.png)<br>
   ![](Res/Pasted%20image%2020260115151434.png)
   
### 3. Настройка папок и ресурсов
1. Структура
```
Assets/ 
|-------Tours/TourName/ 
|--------------Scripts/
|--------------Scenes/
|--------------DLL/ <--- папка для хранения .dll.bytes, создается автоматически
|--------------Resources/ 
|---------------------Prefabs/ 
|---------------------Materials/ 
|---------------------Fonts/ 
|---------------------Audios/ 
|---------------------и т.д
```

2. Использование сцен
   В проект возможно использовать только одну сцену, переходов в другие не должно быть
# Создание тура

1. Точка входа
   Необходимо иметь класс, зависимый от MonoBehaviour, который будет точкой входа в тур
```c#
namespace TourName
{
	internal sealed class EntryPoint : MonoBehaviour 
	{ 
		   [SerializeField] private MonoClass monoClass; 
		   private SharpClass sharpClass; 
		   [SerializeField] private float floatValue; 
		   
		   private void Awake 
			{ 
			   monoClass.Init(floatValue); 
			   sharpClass = new(monoClass); 
			} 
	}
}
```

2. Камера игрока
   На камеру игрока необходимо добавить компонент TourverseToolkit.PlayerCamera
   
   ![](Res/Pasted%20image%2020260115155136.png)<br>
   
4. Внутри точки входа необходимо вызвать в методе Start методы из TourverseToolkit
   TourController.TourStart(PlayerCamera);
   
   ![](Res/Pasted%20image%2020260115154809.png)<br>
6. Отслеживание успешного запуска
   По дефолту достаточно вызвать спустя 30 секунд TourController.CheckPoint(int) - указав контрольную точку 1
   
   ![](Res/Pasted%20image%2020260115155514.png)<br>
   Так же можно сделать несколько контрольных точек, например на квесты, 1квест = 1 контрольная точка

8. Пространство имен<br>
   Каждый класс должен иметь пространство имен, которое соответствует названию тура
```c#
namespace TourName
{
}
```

Для удобства можно назначить Root Namespace поле в AssemblyDefinition

![](Res/Pasted%20image%2020260115152958.png)<br>
**Имейте ввиду, что назначение этого поля не означает выставление namespace во всех уже существующих классах

3. Генерация файлов
   По завершению разработки тура необходимо сгенерировать Addressables файлы. Tourverse --> Settinngs --> Generate
   
   ![](Res/Pasted%20image%2020260115153159.png)<br>
   В Addressables Groups должно получиться так:
   
   ![](Res/Pasted%20image%2020260115153319.png)<br>
**Наличие Labels обязательно

5. Сборка файлов в архив
   По пути [ProjectName/ServerData] - хранятся собранные туры. Собираем их в .7z и передаем публицисту.
   
   ![](Res/Pasted%20image%2020260115153548.png)<br>
   
   5. link.xml
   Вместе с .7z тура, нужно передать файл [ProjectName/Assets/HybridCLRGenerate/link.xml]
   Пример:
```xml
<?xml version="1.0" encoding="utf-8"?>
<linker>
	<assembly fullname="UnityEngine.CoreModule">
		<type fullname="UnityEngine.Color" preserve="all" />
		<type fullname="UnityEngine.Component" preserve="all" />
		<type fullname="UnityEngine.Coroutine" preserve="all" />
		<type fullname="UnityEngine.DynamicGI" preserve="all" />
		<type fullname="UnityEngine.Events.UnityAction`1" preserve="all" />
		<type fullname="UnityEngine.Events.UnityEvent`1" preserve="all" />
		<type fullname="UnityEngine.GameObject" preserve="all" />
		<type fullname="UnityEngine.HeaderAttribute" preserve="all" />
		<type fullname="UnityEngine.Material" preserve="all" />
		<type fullname="UnityEngine.Mathf" preserve="all" />
		<type fullname="UnityEngine.MonoBehaviour" preserve="all" />
		<type fullname="UnityEngine.Object" preserve="all" />
		<type fullname="UnityEngine.PlayerPrefs" preserve="all" />
		<type fullname="UnityEngine.Quaternion" preserve="all" />
		<type fullname="UnityEngine.Random" preserve="all" />
		<type fullname="UnityEngine.Renderer" preserve="all" />
		<type fullname="UnityEngine.RendererExtensions" preserve="all" />
		<type fullname="UnityEngine.RenderSettings" preserve="all" />
		<type fullname="UnityEngine.RequireComponent" preserve="all" />
		<type fullname="UnityEngine.SerializeField" preserve="all" />
		<type fullname="UnityEngine.Sprite" preserve="all" />
		<type fullname="UnityEngine.Time" preserve="all" />
		<type fullname="UnityEngine.Transform" preserve="all" />
		<type fullname="UnityEngine.Vector3" preserve="all" />
		<type fullname="UnityEngine.WaitForSeconds" preserve="all" />
		<type fullname="UnityEngine.TextAreaAttribute" preserve="all" />
		<type fullname="UnityEngine.AnimationCurve" preserve="all" />
		<type fullname="UnityEngine.Application" preserve="all" />
		<type fullname="UnityEngine.Behaviour" preserve="all" />
		<type fullname="UnityEngine.Camera" preserve="all" />
		<type fullname="UnityEngine.Color32" preserve="all" />
		<type fullname="UnityEngine.Cursor" preserve="all" />
		<type fullname="UnityEngine.CursorLockMode" preserve="all" />
		<type fullname="UnityEngine.Keyframe" preserve="all" />
		<type fullname="UnityEngine.LayerMask" preserve="all" />
		<type fullname="UnityEngine.MeshRenderer" preserve="all" />
		<type fullname="UnityEngine.RangeAttribute" preserve="all" />
		<type fullname="UnityEngine.SceneManagement.SceneManager" preserve="all" />
		<type fullname="UnityEngine.Shader" preserve="all" />
		<type fullname="UnityEngine.SkinnedMeshRenderer" preserve="all" />
		<type fullname="UnityEngine.Space" preserve="all" />
		<type fullname="UnityEngine.SpaceAttribute" preserve="all" />
		<type fullname="UnityEngine.TooltipAttribute" preserve="all" />
		<type fullname="UnityEngine.Vector2" preserve="all" />
		<type fullname="UnityEngine.Vector4" preserve="all" />
	</assembly>

	<assembly fullname="UnityEngine.UI">
		<type fullname="UnityEngine.UI.Image" preserve="all" />
		<type fullname="UnityEngine.UI.Slider" preserve="all" />
		<type fullname="UnityEngine.UI.Slider/SliderEvent" preserve="all" />
		<type fullname="UnityEngine.UI.Text" preserve="all" />
	</assembly>
	
	<assembly fullname="mscorlib">
		<type fullname="System.Array" preserve="all" />
		<type fullname="System.Byte" preserve="all" />
		<type fullname="System.Collections.Generic.IEnumerator`1" preserve="all" />
		<type fullname="System.Collections.IEnumerator" preserve="all" />
		<type fullname="System.Diagnostics.DebuggableAttribute" preserve="all" />
		<type fullname="System.Diagnostics.DebuggableAttribute/DebuggingModes" preserve="all" />
		<type fullname="System.Diagnostics.DebuggerHiddenAttribute" preserve="all" />
		<type fullname="System.IDisposable" preserve="all" />
		<type fullname="System.Int32" preserve="all" />
		<type fullname="System.NotSupportedException" preserve="all" />
		<type fullname="System.Object" preserve="all" />
		<type fullname="System.Runtime.CompilerServices.CompilationRelaxationsAttribute" preserve="all" />
		<type fullname="System.Runtime.CompilerServices.CompilerGeneratedAttribute" preserve="all" />
		<type fullname="System.Runtime.CompilerServices.IteratorStateMachineAttribute" preserve="all" />
		<type fullname="System.Runtime.CompilerServices.RuntimeCompatibilityAttribute" preserve="all" />
		<type fullname="System.Runtime.CompilerServices.RuntimeHelpers" preserve="all" />
		<type fullname="System.RuntimeFieldHandle" preserve="all" />
		<type fullname="System.String" preserve="all" />
		<type fullname="System.Type" preserve="all" />
		<type fullname="System.ValueType" preserve="all" />
		<type fullname="System.Action`1" preserve="all" />
		<type fullname="System.AsyncCallback" preserve="all" />
		<type fullname="System.Collections.Generic.List`1" preserve="all" />
		<type fullname="System.Delegate" preserve="all" />
		<type fullname="System.IAsyncResult" preserve="all" />
		<type fullname="System.MulticastDelegate" preserve="all" />
		<type fullname="System.Threading.Interlocked" preserve="all" />
		<type fullname="System.Action" preserve="all" />
		<type fullname="System.Enum" preserve="all" />
	</assembly>
</linker>

```

6. Использование сторонних библиотек
   Для использования сторонних библиотек необходимо сделать запрос на установку ее в платформу, для возможности использования функционала платформой.

# Загрузка на платформу Tourverse

# Возможные проблемы и решения

# Часто задаваемые вопросы

# Приложение
