# access_ptr: Memory accessors that look like Rust pointers for C++

Experimeting with Rust give me a new view on programming. 
One interisting aspect is that Rust's raw pointers aren't changed with arithmetic operators (`++`, `--`, `+`, `-`). 
This makes code that iterate over many pointers more readable, as we don't get confused by if the characters means an arithmetic or a pointer operation.
Therefore, this small library aims to offer Rust like pointers for C++ for -- IMHO -- more readable code.
The name accessor comes from Pytorch's C++ library, besides Rust like pointers, this library adds a second optional paramters specifying the stride size of the elements.

The following code uses C++'s plain pointers:

```C++
uint8_t * const in_rgb_ptr = in_image.ptr<uint8_t>();
float * const mask_ptr = mask.ptr<float>();
uint8_t *out_rgb_ptr = out_image.ptr<uint8_t>();

for (int i = 0; i<width*height; ++i) {
    const float mask_value = *mask_ptr++;
    out_rgb_ptr[0] = in_rgb_ptr[0] * mask_value;
    out_rgb_ptr[1] = in_rgb_ptr[1] * mask_value;
    out_rgb_ptr[2] = in_rgb_ptr[2] * mask_value;

    in_rgb_ptr += 3;
    out_rgb_ptr += 3;
}
```

it's a small code, but I found the pointer's `+=`, `*` distracting.
The `const`ness of the pointers is hard to track also.
Now consider using a more rustacean approach:

```C++
// Plain accessor
notstd::accessor_ptr<const float> mask_ptr = mask.ptr<float>();
// The `3` means increment by sizeof(uint8_t)*3. 
notstd::accessor_ptr<const uint8_t, 3> in_rgb_ptr = in_image.ptr<uint8_t>();
notstd::accessor_ptr<uint8_t, 3> out_rgb_ptr = out_image.ptr<uint8_t>();

for (int i = 0; i<width*height; ++i) {
    // No deference operator.
    const float mask_value = mask_ptr.value();
    
    out_rgb_ptr[0] = in_rgb_ptr[0] * mask_value;
    out_rgb_ptr[1] = in_rgb_ptr[1] * mask_value;
    out_rgb_ptr[2] = in_rgb_ptr[2] * mask_value;

    // Will actually increment by 3
    in_rgb_ptr = in_rgb_ptr.next(); 
    out_rgb_ptr = out_rgb_ptr.next();
    mask_ptr = mask_ptr.next()
}
```
