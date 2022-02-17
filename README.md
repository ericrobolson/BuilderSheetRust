# BuilderSheetRust

This is a runtime library for using sprite sheets generated from https://github.com/ericrobolson/BuilderGenerator.

Checkout the `example` folder in https://github.com/ericrobolson/BuilderGenerator/tree/main/example for client side usage.

## Usage
### Loading
```
let sprite_sheet = {
	let mut file = filesystem::open(ctx, "/renders/test_cube.json").unwrap();

	let mut buffer = Vec::new();
	file.read_to_end(&mut buffer)?;

	let json = std::str::from_utf8(&buffer).unwrap();

	SpriteSheet::from_json(&json).unwrap()
};

let animation = sprite_sheet.animations()[0].name().to_string();
let direction = 0;
let frame = 0;

let image = graphics::Image::from_bytes(ctx, &sprite_sheet.png_bytes())?;
```

### Getting images
```
if let Some(animation) = self.sprite_sheet.animation(&self.animation) {
// Increment direction
if self.frame as usize >= animation.direction(self.direction).unwrap().frames().len() {
	self.frame = 0;
	self.direction += 1;
}

// Increment animation
if self.direction as usize >= animation.directions().len() {
	self.direction = 0;
	self.animation_idx += 1;

	// Wrap animations
	if self.animation_idx >= self.sprite_sheet.animations().len() {
		self.animation_idx = 0;
	}

	self.animation = self.sprite_sheet.animations()[self.animation_idx]
		.name()
		.to_string();
}

println!("dir: {:?}", self.direction);
}
```

### Example Render
```
graphics::clear(ctx, [0.1, 0.2, 0.3, 1.0].into());

let color = Color::from((255, 255, 255));
let dest_point = glam::Vec2::new(0.0, 0.0);
let param = {
	let mut param: graphics::DrawParam = (dest_point, 0.0, color).into();

	if let Some(animation) = self.sprite_sheet.animation(&self.animation) {
		let direction = animation.direction(self.direction).unwrap();
		let frame = direction.frame(self.frame).unwrap();

		// Do subsprite
		param = param.src(graphics::Rect::new(
			frame.start_x_normalized() as f32,
			frame.start_y_normalized() as f32,
			frame.width_normalized() as f32,
			frame.height_normalized() as f32,
		));

		// Do offset
		let offset =
			glam::Vec2::new(frame.offset_x_px() as f32, frame.offset_y_px() as f32);
		param = param.dest(offset);
	}

	param
};

graphics::draw(ctx, &self.image, param)?;

graphics::present(ctx)?;
```
