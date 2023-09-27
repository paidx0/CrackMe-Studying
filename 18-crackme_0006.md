![image.png](https://cdn.nlark.com/yuque/0/2023/png/22837360/1695805475261-3628c205-4915-4468-b1a3-8debb0a1d99f.png#averageHue=%23eaeaea&clientId=uf2088880-3076-4&from=paste&height=116&id=u4f68e384&originHeight=232&originWidth=560&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=21738&status=done&style=none&taskId=ueb90b822-c844-466b-a2fa-1e2496aa5ec&title=&width=280.3333435058594)<br />找到函数入口<br />前面的计算c盘 d盘序列号左移操作，都是多余的，关键处是开根号值<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22837360/1695806731280-f6eb3d21-6963-4da8-9939-4afea82fb9e0.png#averageHue=%23fdfcfc&clientId=uf2088880-3076-4&from=paste&height=302&id=u7d04269b&originHeight=712&originWidth=1344&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=146574&status=done&style=none&taskId=ua75fba92-971b-4c89-a10e-bfa2dce2253&title=&width=569.3333740234375)<br />累乘函数<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22837360/1695805932588-3eed4751-9ddf-42d9-963a-cd086b3c58fd.png#averageHue=%2311110e&clientId=uf2088880-3076-4&from=paste&height=174&id=u1fce6536&originHeight=445&originWidth=1074&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=71644&status=done&style=none&taskId=ue21f2e0b-8679-47be-ab35-87e199c4e7c&title=&width=420)<br />验证<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22837360/1695808611017-61127f85-a4e1-4f4e-a586-3ce2585636c3.png#averageHue=%23efefee&clientId=u51de2be3-c179-4&from=paste&height=159&id=ud78a4544&originHeight=285&originWidth=549&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=35432&status=done&style=none&taskId=u2f65f5cf-28d8-4e4e-9c29-26472ee8359&title=&width=307)![image.png](https://cdn.nlark.com/yuque/0/2023/png/22837360/1695808759472-be6f5a33-4994-4e7e-a034-327aaf5b777a.png#averageHue=%232d2c2b&clientId=u51de2be3-c179-4&from=paste&height=105&id=u6cec115d&originHeight=218&originWidth=727&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=40281&status=done&style=none&taskId=u45b31b5b-46a6-4761-825d-8050c5e6267&title=&width=348.66668701171875)
```cpp
#include <iostream>
#include <atlstr.h>

// 获取盘序列号
DWORD GetVolumeSerialNumber(const char* lpRootPathName) {
    CHAR FileSystemNameBuffer[128];
    DWORD FileSystemFlags;
    DWORD VolumeSerialNumber;
    CHAR VolumeNameBuffer[128];
    GetVolumeInformationA(
        lpRootPathName,
        VolumeNameBuffer,
        0x80u,
        &VolumeSerialNumber,
        (LPDWORD)0xFF,
        &FileSystemFlags,
        FileSystemNameBuffer,
        0x80u);
    return VolumeSerialNumber;
}
// 计算平方开根号
DWORD floatdeal(DWORD a1, DWORD a2) {
    int res = 0;
    __asm {
        fwait;
    fninit;
    fild dword ptr[a1];
    fld st(0);
    fmulp st(1), st(0);
    fild dword ptr[a2];
    fld st(0);
    fmulp st(1), st(0);
    faddp st(1), st(0);
    fsqrt;
    fistp dword ptr[res];
}
return res;
}
// 左移
DWORD bit_move(DWORD val, int n) {
    DWORD size = sizeof(val) * 8;
    n = n % size;
    return (val >> (size - n) | (val << n));
}

int main()
{
    DWORD volumeSerialNumber_C = 0;
    DWORD volumeSerialNumber_D = 0;
    DWORD tmp = 0;
    DWORD tmp2 = 1;
    LONGLONG tmp2_1 = 1;
    char serial[20] = { 0 };
    const char* arr = "071362de9f8ab45c";

    char name[] = "paidx";
    if (strlen(name) < 4) return 0;
    // 得到开根号值
    volumeSerialNumber_C = GetVolumeSerialNumber("C:\\");
    volumeSerialNumber_D = GetVolumeSerialNumber("D:\\");
    tmp = floatdeal(volumeSerialNumber_C, volumeSerialNumber_D);
    // 累乘，溢出值再相加
    for (int i = 0; name[i]; i++)
        {
            tmp2_1 *= name[i];
            tmp2 = tmp2_1 & 0x00000000FFFFFFFF;	// 低32位
            tmp2 += (tmp2_1 & 0xffffffff00000000) >> 32; // 溢出的高32位
        }
    // 左移1位，和开根号值或运算，再与运算
    tmp2 = bit_move(tmp2, 1);
    tmp2 |= tmp;
    tmp2 &= 0x0FFFFFFF;
    // 取模，从索引取值保存
    DWORD i = 0;
    do
        {
            serial[i++] = arr[tmp2 % 0x10];
            tmp2 /= 4;
        } while (tmp2);

    std::cout << serial << std::endl;
}
```
