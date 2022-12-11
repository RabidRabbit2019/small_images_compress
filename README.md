## small_images_compress

1. Utility for convert BGR (24 bits per pixel) or BGRA (32 bits per pixel) TGA images to compressed R5G5B5 pixels (two byte per pixel) data, stored as uint8_t array in .h and .cpp files.
2. Code for decompess that data to R5G5B5 (or R5G6R5) image.

### notice
  I am use this utility/code for project with ST7735 based 160x128 screen. Uncompressed R5G6R5 data has size 11120 bytes, compressed data 3758 bytes, it saves 7362 bytes.

## Build convert utility
  g++ zic_utils.cpp tga_2_src.cpp -O2 -s -Wall -Wextra -o tga_2_src

## Using convert utility
  ./tga_2_src ~/Images/temp_plus.tga temp_plus.h temp_plus.cpp

## Using decompress in code
  example for https://github.com/afiskon/stm32-st7735 library
```c++
void ST7735_DrawImage(int x, int y, int w, int h, const uint8_t * a_data, int a_data_len) {
  // screen wide line buffer
  uint8_t v_image_line[ST7735_WIDTH * sizeof(uint16_t)];

  // check
  if((x >= ST7735_WIDTH) || (y >= ST7735_HEIGHT)) return;
  if((x + w - 1) >= ST7735_WIDTH) return;
  if((y + h - 1) >= ST7735_HEIGHT) return;

  // bytes in one image row
  uint32_t v_row_bytes = w * sizeof(uint16_t);

  // decompress state struct
  zic_decompress_state_s v_st;
  // initialize it
  zic_decompress_init( a_data, a_data_len, v_image_line, w, h, v_st );

  // prepare for write window data
  ST7735_Select();
  ST7735_SetAddressWindow((uint8_t)x, (uint8_t)y, (uint8_t)(x+w-1), (uint8_t)(y+h-1));

  // once set up data kind (D/C input of ST7735 screen)
  GPIOB->BSRR = GPIO_BSRR_BS14;
  
  // for each row
  for ( ; h > 0; --h ) {
    // decompress row
    if ( zic_decompress_row( v_st ) ) {
      // write row bytes to SPI
      ST7735_spi_write( v_image_line, v_row_bytes );
    }
  }

  // release screen interface
  ST7735_Unselect();
}
```
