---
title: Win UI 3
tags:
  - windows
  - cpp/winrt
---
## 标题栏  
```cpp
AppWindow().Resize({ 400, 200 });

auto appWindowPresenter = AppWindow().Presenter().as<OverlappedPresenter>();
appWindowPresenter.IsResizable(false);
appWindowPresenter.SetBorderAndTitleBar(false, false);
this->ExtendsContentIntoTitleBar(true);
```

## 退出  
```cpp
Application::Current().Exit();
```

## 式样  
```xaml
<Button Click="Button_Click" 
		 Background="Red" Foreground="White"
		Margin="0,10,0,0" Grid.Row="3" HorizontalAlignment="Center">
	<Button.Style>
		<Style TargetType="Button" BasedOn="{StaticResource DefaultButtonStyle}">
			<Setter Property="UseSystemFocusVisuals" Value="False"/>
			<Setter Property="Content" Value="Stop"/>
		</Style>
	</Button.Style>
</Button>
```