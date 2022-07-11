// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © loxx

//@version=5
indicator("STD Aadaptive, floating RSX Dynamic Momentum Index [Loxx]", shorttitle = "SADMIFL [Loxx]", 
     overlay = false, 
     timeframe="", 
     timeframe_gaps = true)

greencolor = #2DD204
redcolor = #D2042D 

_rsx_rsi(src, len)=>
    src_out = 100 * src
    mom0 = ta.change(src_out)
    moa0 = math.abs(mom0)
    Kg = 3 / (len + 2.0)
    Hg = 1 - Kg
 
    //mom
    f28 = 0.0, f30 = 0.0
    f28 := Kg * mom0 + Hg * nz(f28[1])
    f30 := Hg * nz(f30[1]) + Kg * f28
    mom1 = f28 * 1.5 - f30 * 0.5
 
    f38 = 0.0, f40 = 0.0
    f38 := Hg * nz(f38[1]) + Kg * mom1
    f40 := Kg * f38 + Hg * nz(f40[1])
    mom2 = f38 * 1.5 - f40 * 0.5
 
    f48 = 0.0, f50 = 0.0
    f48 := Hg * nz(f48[1]) + Kg * mom2
    f50 := Kg * f48 + Hg * nz(f50[1])
    mom_out = f48 * 1.5 - f50 * 0.5
 
    //moa
    f58 = 0.0, f60 = 0.0
    f58 := Hg * nz(f58[1]) + Kg * moa0
    f60 := Kg * f58 + Hg * nz(f60[1])
    moa1 = f58 * 1.5 - f60 * 0.5
 
    f68 = 0.0, f70 = 0.0
    f68 := Hg * nz(f68[1]) + Kg * moa1
    f70 := Kg * f68 + Hg * nz(f70[1])
    moa2 = f68 * 1.5 - f70 * 0.5
 
    f78 = 0.0, f80 = 0.0
    f78 := Hg * nz(f78[1]) + Kg * moa2
    f80 := Kg * f78 + Hg * nz(f80[1])
    moa_out = f78 * 1.5 - f80 * 0.5
 
    rsiout = math.max(math.min((mom_out / moa_out + 1.0) * 50.0, 100.00), 0.00)
    rsiout
    
_volatilityRatio(src) =>
    price = src
    price2 = price * price
    sum = 0., sum2 = 0., sumd = 0.
    m_period = 0., deviation = 0.
    if (bar_index > m_period) 
        sum := nz(sum[1]) + price - nz(price[m_period])
        sum2 := nz(sum2[1]) + price2 - nz(price2[m_period])
    else 
        sum := price
        sum2 := price2
        for k = 1 to m_period
            sum += nz(price[k])
            sum2 += nz(price2[k])
    deviation := (math.sqrt((sum2 - sum * sum / m_period) / m_period))
    if (bar_index > m_period)
        sumd := sumd + deviation - nz(deviation[m_period])
    else 
        sumd := deviation
        for k = 1 to m_period
            sumd += nz(deviation[m_period])
    out = (sumd > 0 ?  m_period * deviation / sumd : 1)
    out
    
src = input.source(close)

dmiper = input.int(15, "DMI Period", group = "Basic Settings")
dmilow = input.int(10, "DMI Lower Limit", group = "Basic Settings")
dmihigh = input.int(50, "DMI Upper Limit", group = "Basic Settings")
devper = input.int(10, "Standard Deviation Period", group = "Basic Settings")
dslper = input.int(10, "Discontinued Signal Lines (DSL) Period", group = "Basic Settings")

colorbars = input.bool(false, "Color bars?", group = "UI Options")

levu = 0., levd = 0., levm = 0.
_dslPeriod = (dslper > 1 ? dslper : 1)

vr = _volatilityRatio(devper) 
td = (dmiper / vr)

if (td > dmihigh)
    td := dmihigh
    
if (td < dmilow)
    td := dmilow

val = _rsx_rsi(src, td)

_alpha = 2.0 / (1.0 + _dslPeriod / vr)

levu := (val >= 50) ? nz(levu[1]) + _alpha * (val - nz(levu[1])) : nz(levu[1])
levd := (val < 50) ? nz(levd[1]) + _alpha * (val - nz(levd[1])) : nz(levd[1])

levm := (levu + levd) / 2.0

colorout = val > levu ? greencolor : val < levd ? redcolor : color.gray
plot(val, color = colorout, linewidth = 3)

plot(levu, color = bar_index % 2 ? color.gray : na)
plot(levm, color = bar_index % 2 ? color.gray : na)
plot(levd, color = bar_index % 2 ? color.gray : na)

barcolor(colorbars ? colorout : na)



























