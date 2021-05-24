Struktura plików:

module/store\
module/store/models\
module/store/effects\
module/store/action-type.ts\
module/store/actions.ts\
module/store/selectors.ts\
module/store/reducers.ts

**State**

Przed utworzeniem obiektu state przygotowujemy model.
State powinien posiadać płaską strukturę:

```typescript
export interface PatientState {
  isPatientModalOpen: boolean;
  patients: Patient[];
}
```

**Akcje**

Opis typu akcji:

[Source] Event – Kontekst kategorii akcji oraz gdzie akcja została zdispatchowana

```typescript
export enum GoogleFitnessActionType {
  FetchFitnessDataSuccess = '[Fitness API] Fetch Fitness Data Success',
}
```

Funkcje akcji:

- do nazwy funkcji dodajemy suffix Action
- propsy grupujemy w obiekcie

```typescript
export const loginAction = createAction(
    '[Login Page] Login',
    props<{ user: User }>()
);
```

**Reducery**

W reducerach używamy pomocniczej funkcji produceOn która jest wrapperem na funkcje on immera

```typescript
const reducer = createReducer<GoogleFitnessState>(
    initialState,
    produceOn(fetchFitnessDataSuccessAction, (draft, payload) => {
        draft.fitnessResults = payload.fitnessResults;
    }),
);
```

**Selectory**

Nazwy selectorów poprzedzamy prefixem select

```typescript
export const selectUser = (state: AppState) => state.selectedUser;
```

**Testy**

**Reducery:**

```typescript
import { initialState as initialPatientState, reducer } from '@app/features/patient/store/reducer';

describe('patient modal', () => {

    it('should have isPatientModalOpen set to true', () => {
      const expectedState: PatientState = { ...initialPatientState, isPatientModalOpen: true };
      expect(reducer(initialPatientState, openAddPatientModalAction)).toEqual(expectedState);
    });

    it('should have isPatientModalOpen set to false', () => {
      const expectedState: PatientState = { ...initialPatientState, isPatientModalOpen: false };
      expect(reducer(initialPatientState, closeAddPatientModalAction)).toEqual(expectedState);
    });

});
```

**Selectory:**
```typescript
import { selectPatientModalIsOpen } from '@app/features/patient/store/selectors';
import { initialState as initialPatientState, KEY } from '@app/features/patient/store/reducer';
import { initialState as initialCoreState } from '@app/core/store/reducer';

describe('Patient selectors tests', () => {

  describe('selectIsPatientModalIsOpen', () => {

    const state = {
      ...initialCoreState,
      [KEY]: initialPatientState
    };

    it('should return expected value using projector', () => {
      expect(selectPatientModalIsOpen.projector(initialPatientState)).toEqual(false);
    });

    it('should return expected value using provided state', () => {
      expect(selectPatientModalIsOpen(state)).toEqual(false);
    });

  });
});
```

**Effekty:**

Do tworzenia testów posiłkujemy się biblioteką jasmine-marbles
https://github.com/ReactiveX/rxjs/blob/master/docs_app/content/guide/testing/marble-testing.md#marble-syntax

```typescript
describe('addPatient$', () => {

    it('should return correct actions stream', () => {
      const mockedPatientId = 5;
      spyOn(patientService, 'addPatient').and.returnValue(of(mockedPatientId));
      const source = cold('a', { a: submitAddPatientFormAction({ patient: patientMock })});
      const effects = new PatientEffects(new Actions(source), patientService, router, store);
      const expected = cold('(abcde)', {
        a: stopBlockUIAction({target: BlockUITargets.AddPatientModal} ),
        b: patientAddSuccessAction({ addedPatientId: mockedPatientId }),
        c: closeAddPatientModalAction(),
        d: showSuccessToastrAction({ message: 'PATIENT.ADD_PATIENT_SUCCESS_MSG' }),
        e: fetchPatientsAction()
      });
      expect(effects.addPatient$).toBeObservable(expected);
    });

    it('should return correct actions stream on error', () => {
      const error = new Error('Http Error');
      spyOn(patientService, 'addPatient').and.returnValue(throwError(error));
      const source = cold('a', { a: submitAddPatientFormAction({ patient: patientMock })});
      const effects = new PatientEffects(new Actions(source), patientService, router, store);
      const expected = cold('(ab)', {
        a: showErrorToastrAction({ message: 'PATIENT.ADD_PATIENT_ERROR_MSG' } ),
        b: stopBlockUIAction({ target: BlockUITargets.AddPatientModal }),
      });
      expect(effects.addPatient$).toBeObservable(expected);
    });

  });
```


