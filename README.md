# WordPress Customizer with Sass

This experimental plugin allows user settings in the WordPress Customizer to automatically compile into css via Sass.

**_Note that WPCSass has not been tested thoroughly, and is not intended for production use at present._**

## How It Works

Once set up, this plugin passes WordPress Customizer variables directly to Sass. It triggers a Sass recompile whenever an option has been changed, with the recompiled css stored in a temporary folder. This allows the new settings to be visible within the customizer preview but not on the live site.

When you click the `Save & Publish` button in the Customizer, the plugin pushes the changed css live by replacing the theme's stylesheet with the newly compiled css. The most recent version of the stylesheet is also saved as a backup.

## Theme Setup

### Default Configuration

By default, the plugin looks for a file located at `theme_directory/sass/style.scss` as its entry point, and stores the temporary compiled css into a folder: `theme_directory/sass_output/`. The files are generated automatically, so for the basic setup you only need to carry out these steps:

1. Add the following subdirectories to your theme's directory:
```
/sass/
/sass_output/
```
2. Create a `style.scss` file in the `/sass/` folder.
3. Add your theme's metadata to the top of the `style.scss` file (either directly or via an `@import` statement).
4. Create your Scss as desired.
5. Add options to the Customizer as described [below](#options).

## Changing settings

You can change several settings to customise the plugin. These are all method calls which

### Set Sass Input Directory

This setting controls the location of the directory which contains your non-compiled `scss` files.

*Default Value*
`stylesheet_directory/sass/`

*Customisation Method*
```php
$wpcsass->set_sass_input_directory( $directory );
```

*Example*
```php
$wpcsass->set_sass_input_directory( get_stylesheet_directory() . '/sass/' );
```

### Set Sass Entry point

This setting controls the entry point filename which the Sass compiler looks for within the *Sass input directory*.

*Default Value*
`style.scss`

*Customization Method*
```php
$wpcsass->set_sass_entry_point( $filename );
```

*Example*
```php
$wpcsass->set_sass_entry_point( 'style.scss' );
```

### Set Sass Output Directory

This setting controls the location of the directory into which the plugin places the generated `css` files.

*Default Value*
`stylesheet_directory/sass_output/`

*Customisation Method*
```php
$wpcsass->set_sass_output_directory( $directory );
```

*Example*
```php
$wpcsass->set_sass_output_directory( get_stylesheet_directory() . '/sass_output/' );
```

### Set Sass Vardump

This setting controls the file location of the Sass vardump file. [Click here for a brief explanation of this file.](#sass-vardump)

*Default Value*
`stylesheet_directory/sass/_customizer_variables.scss`

*Customisation Method*
```php
$wpcsass->set_sass_vardump( $file );
```

*Example*
```php
$wpcsass->set_sass_vardump( get_stylesheet_directory() . '/sass/_customizer_variables.scss' );
```

### Set Sass Output

This setting controls the file location of the main output file which the plugin generates.

*Default Value*
`stylesheet_directory/sass_output/style.css`

*Customisation Method*
```php
$wpcsass->set_sass_output( $file );
```

*Example*
```php
$wpcsass->set_sass_output( get_stylesheet_directory() . '/sass_output/style.css' );
```

### Set Live CSS location

This setting controls the location of the live css file which the plugin overwrites when saving.

*Default Value*
`stylesheet_directory/style.css`

*Customisation Method*
```php
$wpcsass->set_live_css_location( $file );
```

*Example*
```php
$wpcsass->set_live_css_location( get_stylesheet_directory() . '/style.css' );
```

## Options

### Basics

To add settings, you need to create a function in your theme's (or plugin's) `functions.php` file, and add that function to the `customize_register` hook with a priority of `1`. This function must accept the `$wp_customize` variable as a parameter, and create an instance of the `WPC_Sass` class.

A blank example of what this should look like

```php
function theme_wpcsass( $wp_customize ) {
	$wpcsass = new WPC_Sass;

	// New settings are added here

}
add_action( 'customize_register', 'theme_wpcsass', 1 );
```

### Add Panel

Adds or updates a Customizer panel.

*Method*
```php
$wpcsass->add_panel( $panel_id, $priority, $title );
```

*Example*
```php
$wpcsass->add_panel( 'custom_panel', 50, 'Custom Panel' );
```

### Add Panels

Adds or updates multiple Customizer panels.

*Method*
```php
$wpcsass->add_panels( $panels );
```

*Example*
```php
$wpcsass->add_panels( array(
	'custom_panel' => array( 50, 'Custom Panel' ),
	'another_custom_panel' => array( 60, 'AnotherCustom Panel' )
) );
```

### Add Section

Adds or updates a Customizer section.

*Method*
```php
$wpcsass->add_section( $section_id, $priority, $title, $panel_id );
```

*Example*
```php
$wpcsass->add_section( 'custom_section', 50, 'Test Section', 'custom_panel' );
```

### Add Sections

Adds or updates multiple Customizer sections.

*Method*
```php
add_sections( $sections );
```

*Example*
```php
$wpcsass->add_sections( array(
	'custom_section' => array( 50, 'Test Section', 'custom_panel' ),
	'another_custom_section' => array( 60, 'Test Section', 'custom_panel' )
) );
```

### Add Setting

Adds or updates an existing setting. See below for which `$data` is required for each type of setting.

*Method*
```php
$wpcsass->add_setting( $setting_id, $data );
```

*Example*
```php
$wpcsass->add_setting( 'colour', array(
	'label'   => 'Colour',
	'section' => 'custom_section',
	'type'    => 'colour',
	'default' => #444
) );
```

### Add Settings

Adds or updates multiple settings.

*Method*
```php
$wpcsass->add_settings( $settings );
```

*Example*
```php
$wpcsass->add_settings( array(
	'colour', array(
		'label'   => 'Colour',
		'section' => 'custom_section',
		'type'    => 'colour',
		'default' => #444
	),
	'text', array(
		'label'   => 'Text',
		'section' => 'custom_section',
		'type'    => 'text',
		'default' => 'something'
	)
) );
```

### Options Usage

Combining all the above, a simple example would be:

```php
function theme_wpcsass( $wp_customize ) {
	$wpcsass = new WPC_Sass;

	$wpcsass->add_panel( 'custom_panel', 50, 'Custom Panel' );
    $wpcsass->add_section( 'custom_section', 50, 'Test Section', 'custom_panel' );
    $wpcsass->add_settings( array(
        'colour', array(
            'label'   => 'Colour',
            'section' => 'custom_section',
            'type'    => 'colour',
            'default' => #444
        ),
        'text', array(
            'label'   => 'Text',
            'section' => 'custom_section',
            'type'    => 'text',
            'default' => 'something'
        )
    ) );

}
add_action( 'customize_register', 'theme_wpcsass', 1 );
```

## Available Controls

In addition to the default controls, the plugin includes several custom controls plus a shortcut for creating a comprehensive set of background controls via a single setting. All of the available controls are described below.

### Presentation

This custom control allow you to add presentational text into the layout of the Customizer. It comes in three flavours: **Title**, **Subtitle**, and **Description**.

![Presentation Controls](http://i.imgur.com/XdTG54h.png)

#### Title

Adds a stylised `<h1>` tag containing a label.

```php
'title_setting' => array(
    'label'   => 'Example Title',
    'section' => 'custom_section',
    'type'    => 'title'
)
```

#### Subtitle

Adds a stylised `<h3>` tag containing a label.

```php
'subtitle_setting' => array(
    'label'   => 'Example Subtitle',
    'section' => 'custom_section',
    'type'    => 'subtitle'
)
```

#### Description

Adds a `<p>` tag containing text.

```php
'description_setting' => array(
    'description'   => 'Lorem ipsum dolor sit amet, consectetur adipiscing elit.',
    'section' => 'custom_section',
    'type'    => 'description'
)
```

### Range

The range control allows you to create a slider with minimum and maximum values. It includes a label which displays the current value. You can also click on the label itself and manually type in the number you want.

![Range Control](http://i.imgur.com/fQzrQqW.png)

```php
'range_setting' => array(
    'label'   => 'Range',
    'section' => 'custom_section',
    'type'    => 'range',
    'default' => '3',
    'units'   => 'px',
    'range'   => array(
        'min'  => '0',
        'max'  => '20',
        'step' => '1'
    )
)
```

### Radio

The radio control adds some cleaner presentation to the typical radio functionality. Options are wrapped intelligently, showing either two or three per line depending on the total number. The labels are styled to match the Customizer's button style, witht the addition of a checkmark on the selected option.

![Radio Control](http://i.imgur.com/LGa1SB9.png)

```php
'radio_setting_3' => array(
    'label'   => 'Radio 3',
    'section' => 'custom_section',
    'type'    => 'radio',
    'choices' => array(
        'a' => 'Option A',
        'b' => 'Option B',
        'c' => 'Option C',
      	'd' => 'Option D'
    )
)
```

### Background Section

The background section is a shorthand which allows you to add multiple connected controls without typing them all in manually.

It includes a colour control, an image control, and several radio controls. The radio controls are only visible when an image has been selected, because they control the `background-repeat`, `background-position`, `background-attachment`, and `background-size` properties.

*Note: the background colour control can use either the default or the alpha colour picker.*

![Background Section](http://i.imgur.com/oWpje8n.png)

```php
'background_setting' => array(
    'label'   => 'Background',
    'section' => 'custom_section',
    'type'    => 'background_section',
    'alpha'   => true,
    'default' => '#444'
)
```

### Alpha Colour

The alpha colour control was created by Braad Martin, [and is available on GitHub](https://github.com/BraadMartin/components/tree/master/customizer/alpha-color-picker). It works almost exactly the same as the standard colour control, but with an added transparency option.

![Alpha Colour Control](http://i.imgur.com/OBTRHFo.png)

```php
'alpha_setting' => array(
    'label'   => 'Alpha Colour',
    'section' => 'custom_section',
    'type'    => 'colour',
    'alpha'   => true,
    'default' => '#444'
)
```

### Standard Controls

The standard Customizer controls are also available.

![Standard Controls](http://i.imgur.com/TP834Nr.png)

#### Colour

```php
'colour_setting' => array(
    'label'   => 'Standard Colour',
    'section' => 'custom_section',
    'type'    => 'colour',
    'default' => '#444'
)
```

#### Text

```php
'text_setting' => array(
    'label'   => 'Text',
    'section' => 'custom_section',
    'type'    => 'text',
    'default' => 'default'
)
```

#### Checkbox

```php
'checkbox_setting' => array(
    'label'   => 'Checkbox',
    'section' => 'custom_section',
    'type'    => 'checkbox'
)
```

#### Select

```php
'select_setting' => array(
    'label'   => 'Select',
    'section' => 'custom_section',
    'type'    => 'select',
    'choices' => array(
        'a' => 'Option A',
        'b' => 'Option B',
        'c' => 'Option C'
    )
)
```

#### Textarea

```php
'textarea_setting' => array(
    'label'   => 'Textarea',
    'section' => 'custom_section',
  	'type'    => 'textarea'
)
```

#### Image

```php
'image_setting' => array(
    'label'   => 'Image',
    'section' => 'custom_section',
    'type'    => 'image'
)
```

## Sass Vardump

The Sass vardump is a generated file which contains every Sass variable and value passed from the Customizer into Sass. The Sass compiler does not normally display the available variables, so this vardump acts as a convenient cheat-sheet for easy reference when developing themes.

It also includes every `title` and `subtitle` custom control as comments, which helps break down the sections appropriately.

Note that the background-section controls are all listed at the end of the file because they are dynamically generated.

The [Comprehensive Example Usage](#comprehensive-example-usage) below will output the following Sass Vardump file:

```css

// Example Title
// Example Subtitle
$range_setting: 10px;
$radio_setting: a;
$radio_setting_2: a;
$radio_setting_3: a;
$radio_setting_4: a;
$alpha_setting: rgba(129,215,66,0.61);
$colour_setting: #1e73be;
$text_setting: default;
$checkbox_setting: true;
$select_setting: a;
$textarea_setting: '';
$image_setting: '';

// Background Sections
$background_setting_color: rgba(129,215,66,0.74);
$background_setting_bgimage: 'http://wpcsass.localhost/wp-content/uploads/2017/05/bippa.jpg';
$background_setting_repeat: no-repeat;
$background_setting_position: center top;
$background_setting_attachment: scroll;
$background_setting_size: cover;

```

## Comprehensive Example Usage

This example adds a custom section and a custom panel, and demonstrates all of the available controls.

```php

function theme_wpcsass( $wp_customize ) {
  $wpcsass = new WPC_Sass;

	$wpcsass->add_panel( 'custom_panel', 50, 'Custom Panel' );
	$wpcsass->add_section( 'custom_section', 50, 'Custom Section', 'custom_panel' );
	$wpcsass->add_settings( array(
		'title_setting' => array(
			'label'   => 'Example Title',
			'section' => 'custom_section',
			'type'    => 'title'
		),
		'subtitle_setting' => array(
			'label'   => 'Example Subtitle',
			'section' => 'custom_section',
			'type'    => 'subtitle'
		),
		'description_setting' => array(
			'description' => 'Lorem ipsum dolor sit amet, consectetur adipiscing elit.',
			'section'     => 'custom_section',
			'type'        => 'description'
		),
		'range_setting' => array(
			'label'   => 'Range',
			'section' => 'custom_section',
			'type'    => 'range',
			'default' => '3',
			'units'   => 'px',
			'range'   => array(
					'min'  => '0',
					'max'  => '20',
					'step' => '1'
			),
		),
		'radio_setting' => array(
			'label'   => 'Radio',
			'section' => 'custom_section',
			'type'    => 'radio',
			'default' => 'a',
			'choices' => array(
				'a' => 'Option A',
				'b' => 'Option B'
			)
		),
		'radio_setting_2' => array(
			'label'   => 'Radio 2',
			'section' => 'custom_section',
			'type'    => 'radio',
			'default' => 'a',
			'choices' => array(
				'a' => 'Option A',
				'b' => 'Option B',
				'c' => 'Option C'
			)
		),
		'radio_setting_3' => array(
			'label'   => 'Radio 3',
			'section' => 'custom_section',
			'type'    => 'radio',
			'default' => 'a',
			'choices' => array(
				'a' => 'Option A',
				'b' => 'Option B',
				'c' => 'Option C',
				'd' => 'Option D'
			)
		),
		'radio_setting_4' => array(
			'label'   => 'Radio 4',
			'section' => 'custom_section',
			'type'    => 'radio',
			'default' => 'a',
			'choices' => array(
				'a' => 'Option A',
				'b' => 'Option B',
				'c' => 'Option C',
				'd' => 'Option D',
				'e' => 'Option E',
				'f' => 'Option F'
			)
		),
		'background_setting' => array(
			'label'   => 'Background',
			'section' => 'custom_section',
			'type'    => 'background_section',
			'alpha'   => true,
			'default' => '#444'
		),
		'alpha_setting' => array(
			'label'   => 'Alpha Colour',
			'section' => 'custom_section',
			'type'    => 'colour',
			'alpha'   => true,
			'default' => '#444'
		),
		'colour_setting' => array(
			'label'   => 'Standard Colour',
			'section' => 'custom_section',
			'type'    => 'colour',
			'default' => '#444'
		),
		'text_setting' => array(
			'label'   => 'Text',
			'section' => 'custom_section',
			'type'    => 'text',
			'default' => 'default'
		),
		'checkbox_setting' => array(
			'label'   => 'Checkbox',
			'section' => 'custom_section',
			'type'    => 'checkbox'
		),
		'select_setting' => array(
			'label'   => 'Select',
			'section' => 'custom_section',
			'type'    => 'select',
			'default' => 'a',
			'choices' => array(
				'a' => 'Option A',
				'b' => 'Option B',
				'c' => 'Option C'
			)
		),
		'textarea_setting' => array(
			'label'   => 'Textarea',
			'section' => 'custom_section',
			'type'    => 'textarea'
		),
		'image_setting' => array(
			'label'   => 'Image',
			'section' => 'custom_section',
			'type'    => 'image'
		)
	) );

}
add_action( 'customize_register', 'theme_wpcsass', 1 );
```
