---
Description: Interrupt Sync Objects
MS-HAID: 'audio.interrupt\_sync\_objects'
MSHAttr: 'PreferredLib:/library/windows/hardware'
title: Interrupt Sync Objects
---

# Interrupt Sync Objects


## <span id="interrupt_sync_objects"></span><span id="INTERRUPT_SYNC_OBJECTS"></span>


The PortCls system driver implements the [IInterruptSync](audio.iinterruptsync) interface for the benefit of miniport drivers. **IInterruptSync** represents an interrupt sync object that synchronizes the execution of a list of interrupt service routines (ISRs) with non-interrupt routines.

Interrupt sync objects provide two key capabilities:

-   Execution of a list of ISRs in response to an interrupt. The sync object is connected to an interrupt source. Each time the interrupt occurs, the sync object executes the ISRs in a specified order according to the selected mode. (See the following description of the three modes.)

-   Execution of routines that are not ISRs. These non-interrupt routines are not connected to the sync object's interrupt. Instead, a non-interrupt routine runs at a time of the caller's choosing. However, the sync object executes the non-interrupt routine synchronously with the object's list of ISRs. In other words, the non-interrupt routine runs to completion before any of the ISRs in the sync object's list begin executing, and vice versa.

An interrupt sync object is flexible in dealing with multiple ISRs. The ISRs reside in a linked list that the sync object traverses at interrupt time. When a miniport driver registers an ISR with a sync object, it specifies whether the ISR should be added to the beginning or end of this list.

A miniport driver calls the [**PcNewInterruptSync**](audio.pcnewinterruptsync) function to create an interrupt sync object. During this call, the driver specifies the manner in which the object is to traverse its list of ISRs at interrupt time. The call supports the three options that are specified by the INTERRUPTSYNCMODE enumeration constants in the following table.

<table>
<colgroup>
<col width="50%" />
<col width="50%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">Constant</th>
<th align="left">Meaning</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left"><p><strong>InterruptSyncModeNormal</strong></p></td>
<td align="left"><p>Call each ISR in the list until one of them returns STATUS_SUCCESS.</p></td>
</tr>
<tr class="even">
<td align="left"><p><strong>InterruptSyncModeAll</strong></p></td>
<td align="left"><p>Call each ISR in the list exactly once, regardless of the return codes of the preceding ISRs.</p></td>
</tr>
<tr class="odd">
<td align="left"><p><strong>InterruptSyncModeRepeat</strong></p></td>
<td align="left"><p>Traverse the entire list of ISRs until a trip through the list occurs in which no ISR in the list returns STATUS_SUCCESS.</p></td>
</tr>
</tbody>
</table>

 

In the **InterruptSyncModeNormal** mode, the sync object calls each ISR in the list until one of them returns STATUS\_SUCCESS. Any ISRs in the list that follow this ISR are not called. This mode emulates the way that the operating system normally handles ISRs. If none of the ISRs return STATUS\_SUCCESS, the behavior is the same as **InterruptSyncModeAll**.

In the **InterruptSyncModeAll** mode, each ISR in the list is called exactly once, regardless of the return codes of the preceding ISRs. This is intended for more primitive hardware where the source of the interrupt is not deterministic, although it might be useful in other situations as well. For example, two interrupt sources might be tightly synchronized on every interrupt, regardless of which of the two sources a particular interrupt comes from.

In the **InterruptSyncModeRepeat** mode, the sync object repeatedly traverses the entire list of ISRs until a trip through the list occurs in which no routine in the list returns STATUS\_SUCCESS. This mode is appropriate for situations in which interrupts from multiple sources might fire on the same interrupt line at the same time, or a second interrupt might fire during ISR processing. Every interrupt source must be able to determine whether it requires processing. The system will stop responding if an ISR that always returns STATUS\_SUCCESS is registered with a sync object in this mode.

In any of these modes, the sync object will acknowledge the interrupt to the operating system if any of the registered ISRs return STATUS\_SUCCESS. In all three modes, if all interrupt sources indicate that they did not successfully handle the interrupt, the sync object will return an unsuccessful result code to the operating system.

The **IInterruptSync** interface supports the following methods:

[**IInterruptSync::CallSynchronizedRoutine**](audio.iinterruptsync_callsynchronizedroutine)

[**IInterruptSync::Connect**](audio.iinterruptsync_connect)

[**IInterruptSync::Disconnect**](audio.iinterruptsync_disconnect)

[**IInterruptSync::GetKInterrupt**](audio.iinterruptsync_getkinterrupt)

[**IInterruptSync::RegisterServiceRoutine**](audio.iinterruptsync_registerserviceroutine)

 

 


--------------------
[Send comments about this topic to Microsoft](mailto:wsddocfb@microsoft.com?subject=Documentation%20feedback%20[audio\audio]:%20Interrupt%20Sync%20Objects%20%20RELEASE:%20%287/14/2016%29&body=%0A%0APRIVACY%20STATEMENT%0A%0AWe%20use%20your%20feedback%20to%20improve%20the%20documentation.%20We%20don't%20use%20your%20email%20address%20for%20any%20other%20purpose,%20and%20we'll%20remove%20your%20email%20address%20from%20our%20system%20after%20the%20issue%20that%20you're%20reporting%20is%20fixed.%20While%20we're%20working%20to%20fix%20this%20issue,%20we%20might%20send%20you%20an%20email%20message%20to%20ask%20for%20more%20info.%20Later,%20we%20might%20also%20send%20you%20an%20email%20message%20to%20let%20you%20know%20that%20we've%20addressed%20your%20feedback.%0A%0AFor%20more%20info%20about%20Microsoft's%20privacy%20policy,%20see%20http://privacy.microsoft.com/en-us/default.aspx. "Send comments about this topic to Microsoft")

