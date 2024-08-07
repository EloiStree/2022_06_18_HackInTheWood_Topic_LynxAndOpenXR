My Hololens 1 way to access the hands:
Require: HoloToolkit

```csharp
// Copyright Microsoft Corporation. All rights reserved.
// Licensed under the MIT License. See LICENSE in the project root for license information.

using HoloToolkit.Unity;
using System;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;
using UnityEngine.XR.WSA.Input;



//Source modified: https://github.com/Microsoft/GalaxyExplorer/blob/master/Assets/Scripts/Input/HandInput.cs
public class HandsInput : SingleInstance<HandsInput>
    {

  

    [System.Serializable]
        public struct HandStateRaw
    {
        [Tooltip("Does the hand is tracked ?")]
        public bool valid;

        [Tooltip("Each Time different")]
        public uint id;


        [Tooltip("Is tapping ?")]
        public bool isTappingDown;

        [Tooltip("Time Since Level Load")]
        public float tapPressedStartTime;
          [Tooltip("From Hololens camera start point")]
            public Vector3 latestTapStartPosition;
        [Tooltip("From Hololens camera start point")]
        public Vector3 latestObservedPosition;
        [Tooltip("From Hololens camera start point")]
        public Vector3 firstObservedPosition;

        public UnityEvent m_onEnter;
            public UnityEvent m_onDown;
            public UnityEvent m_onUp;
            public UnityEvent m_onExit;

      
    }

        public  HandStateRaw[] handStates = new HandStateRaw[2];

   
        private enum QueuedActionType
        {
            HandEnter,
            HandExit,
            HandMoved,
            TapPressed,
            TapReleased
        }

        private struct QueuedAction
        {
            public QueuedActionType actionType;
            public uint id;
            public Vector3 position;
            public float timestamp;
        }

        private Queue<QueuedAction> actionQueue = new Queue<QueuedAction>();

        public bool HandsInFOV
        {
            get
            {
                for (int i = 0; i < handStates.Length; ++i)
                {
                    if (handStates[i].valid)
                    {
                        return true;
                    }
                }

                return false;
            }
        }

        public event System.Action<bool> HandInFOVChanged;

        protected void Awake()
        {
            //if (MyAppPlatformManager.Platform != MyAppPlatformManager.PlatformId.HoloLens)
            //{
            //    DestroyObject(this);
            //    return;
            //}
            // Start out with a clean state by resetting both hand structures.
            for (int i = 0; i < handStates.Length; ++i)
            {
                UnregisterHandByIndex(i);
            }
        }

        private bool eventsAreRegistered = false;

        private void TryToRegisterEvents()
        {
            if (!eventsAreRegistered)
            {
                InteractionManager.InteractionSourceDetected += OtherThread_OnInteractionSourceDetected;
                InteractionManager.InteractionSourceUpdated += OtherThread_OnInteractionSourceUpdated;
                InteractionManager.InteractionSourceLost += OtherThread_OnInteractionSourceLost;
                InteractionManager.InteractionSourcePressed += OtherThread_InteractionSourcePressed;
                InteractionManager.InteractionSourceReleased += OtherThread_InteractionSourceReleased;
                eventsAreRegistered = true;
            }
        }

        private void TryToUnregisterEvents()
        {
            if (eventsAreRegistered)
            {
                InteractionManager.InteractionSourceDetected -= OtherThread_OnInteractionSourceDetected;
                InteractionManager.InteractionSourceUpdated -= OtherThread_OnInteractionSourceUpdated;
                InteractionManager.InteractionSourceLost -= OtherThread_OnInteractionSourceLost;
                InteractionManager.InteractionSourcePressed -= OtherThread_InteractionSourcePressed;
                InteractionManager.InteractionSourceReleased -= OtherThread_InteractionSourceReleased;
                eventsAreRegistered = false;
            }
        }

        private void Start()
        {
            //if (PlayerInputManager.Instance == null)
            //{
            //    Debug.LogWarning("No active PlayerInputManager found. Not all HandInput functionality will be available.");
            //}

            TryToRegisterEvents();
        }

        protected override void OnDestroy()
        {
            TryToUnregisterEvents();

            // Reset active state for both hands.
            for (int i = 0; i < handStates.Length; ++i)
            {
                UnregisterHandByIndex(i);
            }
            base.OnDestroy();
        }

        private void OnEnable()
        {
            TryToRegisterEvents();
        }

        private void OnDisable()
        {
            TryToUnregisterEvents();
        }

        private void Update()
        {
            lock (actionQueue)
            {
                while (actionQueue.Count > 0)
                {
                    QueuedAction queuedAction = actionQueue.Dequeue();

                    switch (queuedAction.actionType)
                    {
                        case QueuedActionType.HandEnter:
                            OnHandEnter(queuedAction.id, queuedAction.position);
                            break;

                        case QueuedActionType.HandExit:
                            OnHandExit(queuedAction.id, queuedAction.position);
                            break;

                        case QueuedActionType.HandMoved:
                            OnHandMoved(queuedAction.id, queuedAction.position, queuedAction.timestamp);
                            break;

                        case QueuedActionType.TapPressed:
                            OnHandTapPressed(queuedAction.id, queuedAction.position);
                            break;

                        case QueuedActionType.TapReleased:
                            OnHandTapReleased(queuedAction.id, queuedAction.position);
                            break;
                    }
                }
            }
        }

        private bool GetHandState(int index, ref HandStateRaw handState)
        {
            if (index >= 0 && index < handStates.Length)
            {
                handState = handStates[index];
                return true;
            }

            return false;
        }

        private void OnHandEnter(uint id, Vector3 position)
        {
            if (HandInFOVChanged != null && !HandsInFOV)
            {
                HandInFOVChanged(true);
            }

            RegisterHand(id, position);
                for (int i = 0; i < 2; i++)
                {
                    if (handStates[i].id == id)
                    {
                        handStates[i].m_onEnter.Invoke();
                    }

                }
         
        }

        private void OnHandExit(uint id, Vector3 position)
        {
            int index = RegisterHand(id, position);
            if (index != -1)
            {
                if (handStates[index].isTappingDown)
                {
                  //  PlayerInputManager.Instance.TriggerTapRelease();
                  
                }
            }

            UnregisterHandById(id);

            if (HandInFOVChanged != null && !HandsInFOV)
            {
                HandInFOVChanged(false);
            }
            for (int i = 0; i < 2; i++)
            {
                if (handStates[i].id == id)
                {
                    handStates[i].m_onExit.Invoke();
                }

            }
    }

        private void OnHandMoved(uint id, Vector3 position, float timestamp)
        {
            RegisterHand(id, position);
        }

        private void OnHandTapPressed(uint id, Vector3 position)
        {
            int index = RegisterHand(id, position);
            if (index != -1)
            {
                handStates[index].isTappingDown = true;
                handStates[index].tapPressedStartTime = Time.time;
                handStates[index].latestTapStartPosition = position;
                for (int i = 0; i < 2; i++)
                {
                    if (handStates[i].id == id)
                    {
                        handStates[i].m_onDown.Invoke();
                    }

                }


            //if (PlayerInputManager.Instance != null)
            //{
            //    PlayerInputManager.Instance.TriggerTapPress();
            //}
        }
            else
            {
                Debug.Log("HandInput.OnHandTapPressed() - RegisterHand() failed!");
            }
        }

        private void OnHandTapReleased(uint id, Vector3 position)
        {
            int index = RegisterHand(id, position);
            if (index != -1)
            {
                if (handStates[index].isTappingDown)
                {
                    handStates[index].isTappingDown = false;
                for (int i = 0; i < 2; i++)
                {
                    if (handStates[i].id == id)
                    {
                        handStates[i].m_onUp.Invoke();
                    }

                }
                //if (PlayerInputManager.Instance != null)
                //{
                //    PlayerInputManager.Instance.TriggerTapRelease();
                //}
            }
            }
            else
            {
                Debug.Log("HandInput.OnHandTapReleased() - RegisterHand() failed!");
            }
        }

        // Hand State utilities
#region Hand Registration
        private int RegisterHand(uint id, Vector3 position)
        {
            int handIndex = GetRegisteredHandIndex(id);

            if (handIndex != -1)
            {
                // Item already exists.  Update its properties.
                handStates[handIndex].latestObservedPosition = position;
            if (handStates[handIndex].firstObservedPosition == Vector3.zero)
                handStates[handIndex].firstObservedPosition = position;
        }
            else
            {
                // New item.  Find the first available slot and add it.
                for (int i = 0; i < handStates.Length; ++i)
                {
                    if (!handStates[i].valid)
                    {
                        handStates[i].valid = true;
                        handStates[i].id = id;
                        handStates[i].isTappingDown = false;
                        handStates[i].tapPressedStartTime = 0.0f;
                        handStates[i].latestTapStartPosition = Vector3.zero;
                        handStates[i].latestObservedPosition = position;
                        handStates[i].firstObservedPosition = position;
                         handIndex = i;

                        break;
                    }
                }

                if (handIndex == -1)
                {
                    Debug.LogError("HandInput failed to find open slot for id " + id + " (existing ids: " + handStates[0].id + " and " + handStates[1].id + ")");
                }
            }

            return handIndex;
        }

        private void UnregisterHandById(uint id)
        {
            for (int i = 0; i < handStates.Length; ++i)
            {
                if (handStates[i].id == id)
                {
                    UnregisterHandByIndex(i);
                    break;
                }
            }
        }

        private void UnregisterHandByIndex(int index)
        {
            if (index >= 0 && index < handStates.Length)
            {
                handStates[index].valid = false;
                handStates[index].id = uint.MaxValue;
                handStates[index].isTappingDown = false;
                handStates[index].latestTapStartPosition = Vector3.zero;
                handStates[index].latestObservedPosition = Vector3.zero;
            }
        }

        private int GetRegisteredHandIndex(uint id)
        {
            int index = -1;

            for (int i = 0; i < handStates.Length; ++i)
            {
                if (handStates[i].valid && handStates[i].id == id)
                {
                    index = i;
                    break;
                }
            }

            return index;
        }

#endregion

#region OtherThread

        // IMPORTANT
        // Everything below this point is executed on a separate thread.  For thread safety,
        // the only code these functions should execute is pushing a task into a synchronized
        // queue for later analysis on the main thread.
        private void OtherThread_OnInteractionSourceDetected(InteractionSourceDetectedEventArgs args)
        {
            Vector3 handPosition;

            if (args.state.source.kind == InteractionSourceKind.Hand && args.state.sourcePose.TryGetPosition(out handPosition))
            {

            //IAMHERE
            Vector3 position = handPosition; //Camera.main.transform.rotation * handPosition;
            QueuedAction newAction = new QueuedAction { actionType = QueuedActionType.HandEnter, id = args.state.source.id, position = position, timestamp = Time.time };

                lock (actionQueue)
                {
                    actionQueue.Enqueue(newAction);
                }
            }
        }

        private void OtherThread_OnInteractionSourceUpdated(InteractionSourceUpdatedEventArgs args)
        {
            Vector3 handPosition;

            if (args.state.source.kind == InteractionSourceKind.Hand && args.state.sourcePose.TryGetPosition(out handPosition))
            {
            //IAMHERE
            Vector3 position = handPosition; //Camera.main.transform.rotation * handPosition;
                QueuedAction newAction = new QueuedAction { actionType = QueuedActionType.HandMoved, id = args.state.source.id, position = position, timestamp = Time.time };

                lock (actionQueue)
                {
                    actionQueue.Enqueue(newAction);
                }
            }
        }

        private void OtherThread_OnInteractionSourceLost(InteractionSourceLostEventArgs args)
        {
            Vector3 handPosition;

            if (args.state.source.kind == InteractionSourceKind.Hand && args.state.sourcePose.TryGetPosition(out handPosition))
            {

            //IAMHERE
            Vector3 position = handPosition; //Camera.main.transform.rotation * handPosition;
            QueuedAction newAction = new QueuedAction { actionType = QueuedActionType.HandExit, id = args.state.source.id, position = position, timestamp = Time.time };

                lock (actionQueue)
                {
                    actionQueue.Enqueue(newAction);
                }
            }
        }

        private void OtherThread_InteractionSourcePressed(InteractionSourcePressedEventArgs args)
        {
            Vector3 handPosition;

            if (args.state.source.kind == InteractionSourceKind.Hand && args.state.sourcePose.TryGetPosition(out handPosition))
            {
                QueuedAction newAction = new QueuedAction { actionType = QueuedActionType.TapPressed, id = args.state.source.id, position = handPosition, timestamp = Time.time };

                lock (actionQueue)
                {
                    actionQueue.Enqueue(newAction);
                }
            }
        }

        private void OtherThread_InteractionSourceReleased(InteractionSourceReleasedEventArgs args)
        {
            Vector3 handPosition;

            if (args.state.source.kind == InteractionSourceKind.Hand && args.state.sourcePose.TryGetPosition(out handPosition))
            {
                QueuedAction newAction = new QueuedAction { actionType = QueuedActionType.TapReleased, id = args.state.source.id, position = handPosition, timestamp = Time.time };

                lock (actionQueue)
                {
                    actionQueue.Enqueue(newAction);
                }
            }
        }

#endregion
    }
```

``` csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class HandsInputPlus : MonoBehaviour
{
    [Header("In")]
    public HandsInput m_handsinput;
    public Transform m_cameraRepresentative;
    public Transform m_startPointOfHandHololens;

    [Header("Result")]
    public HandInfo m_left= new HandInfo();
    public HandInfo m_right= new HandInfo();
    public HandInfo[] m_hands = new HandInfo[] { new HandInfo(), new HandInfo() };

    [System.Serializable]
    public class HandInfo
    {
        public bool m_isTracked;
        public bool m_isTapping;
        public HandSideType m_handType;
        public float m_timeSinceTapping;
        public ConvertedPosition m_lastPositionDetected;
        public ConvertedPosition m_startTappingPositionDetected;
        public ConvertedPosition m_firstDetectedPosition;

        public HandSideType GuessHandType()
        {
            if (!m_isTracked)
                return HandSideType.Unknow;
            if (m_firstDetectedPosition.m_unityCameraLocalPosition == Vector3.zero)
                return HandSideType.Unknow;
            return m_firstDetectedPosition.m_unityCameraLocalPosition.x >= 0f ? HandSideType.Right : HandSideType.Left;
        }

        internal void Reset()
        {
            m_isTracked = false;
            m_isTapping = false;
            m_handType = HandSideType.Unknow;
            m_timeSinceTapping = 0;
            m_lastPositionDetected.Reset();
            m_firstDetectedPosition.Reset();
            m_firstDetectedPosition.Reset();
        }
    }
    public struct ConvertedPosition
    {
        public bool HasBeenSet() { return m_rawPosition != Vector3.zero; }

        internal void Reset()
        {
            m_rawPosition = Vector3.zero;
            m_unityWorldPosition = Vector3.zero;
            m_unityCameraLocalPosition = Vector3.zero;
        }

        public Vector3 m_rawPosition;
        public Vector3 m_unityWorldPosition;
        public Vector3 m_unityCameraLocalPosition;

    }
    public enum HandSideType { Unknow, Left, Right }


    public void Update()
    {
        if (m_handsinput.handStates.Length < 2)
            return;

        RefreshDataWith(m_handsinput.handStates[0], 0);
        RefreshDataWith(m_handsinput.handStates[1], 1);


        AddTimeToTapping(0);
        AddTimeToTapping(1);

        if (m_left.GuessHandType() != HandSideType.Left)
            m_left = new HandInfo();
        if (m_right.GuessHandType() != HandSideType.Right)
            m_right = new HandInfo();
    }

    private void AddTimeToTapping(int index)
    {
        if (m_hands[index].m_isTapping)
            m_hands[index].m_timeSinceTapping += Time.deltaTime;
        else m_hands[index].m_timeSinceTapping = 0;
    }

    private void RefreshDataWith(HandsInput.HandStateRaw rawHand, int index)
    {
        bool wasTracked = m_hands[index].m_isTracked, isTracked = rawHand.valid;


        if (rawHand.valid)
        {
            m_hands[index].m_isTracked = rawHand.valid;
            m_hands[index].m_isTapping = rawHand.isTappingDown;
            m_hands[index].m_startTappingPositionDetected = ConvertedRawPositionToUnityPosition(rawHand.latestTapStartPosition);
            m_hands[index].m_lastPositionDetected = ConvertedRawPositionToUnityPosition(rawHand.latestObservedPosition);
            m_hands[index].m_firstDetectedPosition = ConvertedRawPositionToUnityPosition(rawHand.firstObservedPosition);
            m_hands[index].m_handType = m_hands[index].GuessHandType();
            if (m_hands[index].m_handType == HandSideType.Left)
                m_left = m_hands[index];
            if (m_hands[index].m_handType == HandSideType.Right)
                m_right = m_hands[index];
      
        }
        else m_hands[index].Reset();



        if (!wasTracked && isTracked)
        {
            NotifyStartTracking(m_hands[index]);
        }
        if (wasTracked && !isTracked)
        {
            NotifyStopTracking(m_hands[index]);
        }


    }

    private ConvertedPosition ConvertedRawPositionToUnityPosition(Vector3 rawPosition)
    {
        ConvertedPosition result;
        result.m_rawPosition = rawPosition;
        result.m_unityWorldPosition = (m_startPointOfHandHololens.rotation* rawPosition) + m_startPointOfHandHololens.position;
        result.m_unityCameraLocalPosition = Quaternion.Inverse(m_cameraRepresentative.rotation) * (result.m_unityWorldPosition - m_cameraRepresentative.position);
        return result;
    }

    private void NotifyStopTracking(HandInfo handInfo)
    {
       // throw new NotImplementedException();
    }

    private void NotifyStartTracking(HandInfo handInfo)
    {
        //throw new NotImplementedException();
    }
}



```
Source: https://gitlab.com/eloistree/2019_04_04_holohandstracking
