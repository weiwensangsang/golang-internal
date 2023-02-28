# Return Multiple Values



The golang function call process realizes parameter passing and return value through **fp**+**offset**, unlike C/C++, which realizes parameter passing and return value through **registers**.



This means that the return list of go can be a continuous area with an indefinite length, so it can naturally return different numbers of values.