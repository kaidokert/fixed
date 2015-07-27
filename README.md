# fixed
fixed point template metaprogramming library

Most Basic usage

    #include "fixed.hpp"
    using namespace fix;
    
    ufixed<3,3> foo; // 3.3 unsigned fixedpoint
    sfixed<3,3> bar; // 3.3 signed fixedpoint
    
    // -13.45 (32 bit) unsigned
    using foo_type = FIXED_RANGE(0, 0.0001, 32); 
    
    // all conversions constexpr
    constexpr foo_type my_val = foo_type::from(0.00004);
    constexpr auto test = my_val.to<double>()
    
    // -12.44 (32 bit) signed
    using bar_type = FIXED_RANGE(-0.0001, 0.0001, 32) bar2;
    
    // Constants
    constexpr auto some_float_constant   = FIXED_CONSTANT(44.5632, 32);
    constexpr auto some_integer_constant = FIXED_CONSTANT_I(563);
    
    // automatic sizing for integer ranges
    FIXED_RANGE_I(-232, 5342) some;

All the types compile down to the fix::fixed template:
    
    template< int I, int F, bool S, typename T, typename RT >
    struct fixed {
        // I:  integer bits (can be negative!)
        // F:  fraction bits (can be negative!)
        // S:  signed?
        // T:  smallest integral type capable of storing the fixed point number
        
        // RT: template instance of value_range< T, T min, T max >
        //     min and max hold the min and max values of this fixed point variable
        //     in fixed point notation
        //     they default to the mins and maxs possible within I, F, and S but if
        //     you provide custom bounds (FIXED_RANGE macro f.e.), they will constrain
        //     the value range further.
        //     All calculations done on the fixed point variables are carried out on
        //     these limits too, so the operations can try to act smart
    };

Supported rounding modes:

    rounding::floor
    rounding::ceil
    rounding::towards_zero
    rounding::towards_infinity
    rounding::nearest_up
    rounding::nearest_down
    // next_even, next_odd not implemented yet

Operations on fixed:
    
    // Shift number up (X > 0) or down (X < 0) and move I and F accordingly.
    auto shifted1 = scaling_shift<X [,rounding mode]>( fixed_point_var ); 
 
    // Shift I and F without touching the data (Ie. data interpretation scaled by 2^^X without losing precision)
    auto shifted2 = virtual_shift<X>( fixed_point_var );
    
    // Shift number up or down, but don't move I and F (Ie. scaled 2^^X)
    auto shifted2 = literal_shift<X [,rounding mode]>( fixed_point_var );
    
    add<[max_size, positive, fits<I [,F], rounding-mode]>( lhs, rhs );
    sub<[max_size, positive, fits<I [,F], rounding-mode]>( lhs, rhs );
    mul<[max_size, positive, fits<I [,F], rounding-mode]>( lhs, rhs );
    div<[max_size, positive, fits<I [,F], rounding-mode]>( lhs, rhs );
    
    // max_size restricts the size of intermediate results
    // fits<I [,F]> promises the result will fit into given typen
    // positive promises the result will be positive
    // rounding-mode sets the rounding-mode if rounding is necessary
    
    // operations generally try minimize precision loss, given the constraints
    // they're working in
    
    operator* (with result-size = max of input sizes, rounding to floor)
    operator+ (with result-size = max of input sizes, rounding to floor)
    operator- (with result-size = max of input sizes, rounding to floor)
    
    // no operator for div given, since any reasonable div operation relies
    // on smart constraints to be useful.
    
for further usage see fixed/fixed_test.cpp
