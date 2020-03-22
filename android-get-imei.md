---
title: 关于getDeviceId和getImei获取值不同的分析
categories: 技术栈
tags:
  - Android
date: 2020-03-22 21:29:09
---
## 问题背景
需要获取imei的值，很容易查询到使用TelephonyManager的getDeviceId()（在API Level 26 以上已标记为@Deprecated，在很多Android 26以上设备上也可继续使用）方法和getImei()方法（注释标记该方法需在API Level 26 以上调用，不过实测在有些26以下设备也可以调用到）   
`Tips:Android Q 以上已关闭获取imei的方法，使用以上两种方法获取均会抛出异常SecurityException，不过本文这不是重点，所以本文指的是Android Q以下相关设备。`  
网上的说法这两个都是获取imei的方法，是使用的API Level不同，所以使用时一直以为如果遇到两个方法都能获取到值的情况，两个值应该相同，都是imei的值。  
近期发现getDeviceId()和getImei()同时可用时获取到的值有时会不同，故跟踪一下源码查看一下区别，在此记录。  

## 问题分析：
跟踪TelephonyManager的getDeviceId()和getImei()实现方法  
<!--more-->

```java
    //android/telephony/TelephonyManager.java
    public String getDeviceId() {
        try {
            ITelephony telephony = getITelephony();
            if (telephony == null)
                return null;
            return telephony.getDeviceId(mContext.getOpPackageName());
        } catch (RemoteException ex) {
            return null;
        } catch (NullPointerException ex) {
            return null;
        }
    }
 	public String getImei() {
        return getImei(getSlotIndex());
    }
     public String getImei(int slotIndex) {
        ITelephony telephony = getITelephony();
        if (telephony == null) return null;
        try {
            return telephony.getImeiForSlot(slotIndex, getOpPackageName(), getFeatureId());
        } catch (RemoteException ex) {
            return null;
        } catch (NullPointerException ex) {
            return null;
        }
    }
```
接下来的工作都是跟踪对应的类调用
getDeviceId():  
PhoneInterfaceManager.java -->getDeviceId(String callingPackage)   
--> getDeviceIdWithFeature(String callingPackage, String callingFeatureId)  
-->  PhoneFactory.getPhone(0).getDeviceId()  
getImei():  
PhoneInterfaceManager.java --> getImeiForSlot(int slotIndex, String callingPackage, String callingFeatureId)  
--> PhoneFactory.getPhone(0).getImei()  
PhoneInterfaceManager 作为TelephonyManager的 server 端，有很多方法，有兴趣的可以再深入了解。
最终获取的是Phone中的值，Phone phone = PhoneFactory.getPhone(0);
frameworks/opt/telephony/src/java/com/android/internal/telephony/Phone.java是一个抽象类,  
对应的子类包括GsmCdmaPhone.java、ImsPhoneBase.java,SipPhoneBase.java,  
通过命名大概能看出是手机应该使用的是GsmCdmaPhone.java,  
也可以看到ImsPhoneBase.java,SipPhoneBase.java 关于getDeviceId()和getImei()的返回值均为null,  
故只需关注GsmCdmaPhone.java即可

```java
    //frameworks/opt/telephony/src/java/com/android/internal/telephony/GsmCdmaPhone.java
    @Override
    public String getDeviceId() {
        if (isPhoneTypeGsm()) {
            return mImei;
        } else {
            CarrierConfigManager configManager = (CarrierConfigManager)
                    mContext.getSystemService(Context.CARRIER_CONFIG_SERVICE);
            boolean force_imei = configManager.getConfigForSubId(getSubId())
                    .getBoolean(CarrierConfigManager.KEY_FORCE_IMEI_BOOL);
            if (force_imei) return mImei;
            String id = getMeid();
            if ((id == null) || id.matches("^0*$")) {
                loge("getDeviceId(): MEID is not initialized use ESN");
                id = getEsn();
            }
            return id;
        }
    }
    @Override
    public String getImei() {
        return mImei;
    }
```
## 结论
**很明显，getImei()是纯粹获取imei值的方法，getDeviceId中包含判断，如果isPhoneTypeGsm() ==false && force_imei = false 也可能获取meid或Esn作为返回值。**
