Creates a tailwind-optimized html table out of technical data listings delimited with colons.

# Example

```
mass: 80kg
height: 3m
width: 10m
speed: 30km/h
```

is converted to:

```html
<table>
	<tbody>
		<tr>
			<td>mass</td>
			<td>80kg</td>
		</tr>
		<tr>
			<td>height</td>
			<td>3m</td>
		</tr>
		<tr>
			<td>width</td>
			<td>10m</td>
		</tr>
		<tr>
			<td>speed</td>
			<td>30km/h</td>
		</tr>
	</tbody>
</table>
```

![App main](/assets/app_main.PNG)
![App preview](/assets/app_preview.PNG)
