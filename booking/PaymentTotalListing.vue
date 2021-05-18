<template>
  <v-row class="content-4">
    <!-- group discount -->
    <v-col
      v-if="booking.getGroupDiscountsArray().length"
      class="col-12 border-bottom py-1"
    >
      <v-row>
        <v-col class="col-12 py-1 font-weight-black">
          {{ $t('paymentDetails.groupDiscounts') }}
        </v-col>
      </v-row>
      <!-- iterate group discount -->
      <v-row
        v-for="(
          groupDiscount, groupDiscountIndex
        ) in booking.getGroupDiscountsArray()"
        :key="groupDiscountIndex"
      >
        <v-col class="col-6 py-1">
          <span class="fal fa-percentage mr-2" />
          {{ groupDiscount.getDescription() }}
        </v-col>
        <v-col class="col-6 py-1">
          <!-- absolute -->
          <span v-if="groupDiscount.getType() === 'absolute'">
            - {{ groupDiscount.getAmount() | currency }}
          </span>
          <span v-else-if="groupDiscount.getType() === 'relative'">
            - {{ groupDiscount.getAmount() }} %
          </span>
        </v-col>
      </v-row>
    </v-col>

    <!-- cause we care -->
    <v-col v-if="booking.getCwc()" class="col-12 py-1 border-bottom">
      <v-row>
        <v-col class="col-3 font-weight-black">
          <v-img :src="cwc" :aspect-ratio="3.1" />
        </v-col>
        <v-col class="col-6 offset-3 align-self-center">
          + {{ booking.getCwc() | currency }}
        </v-col>
      </v-row>
    </v-col>

    <!-- vats -->
    <v-col class="col-12 py-1 border-bottom">
      <!-- total without vats -->
      <v-row class="font-weight-black">
        <v-col class="col-6 py-1">
          {{ $t('adminBasket.netWithoutVat') }}
        </v-col>
        <v-col class="col-6 py-1">
          {{ booking.getTotalWithoutVats() | currency }}
        </v-col>
      </v-row>

      <!-- iterate vats -->
      <v-row v-for="(vat, vatIndex) in booking.getVatsArray()" :key="vatIndex">
        <v-col class="col-6 py-1"> {{ vat.getRate() }} % </v-col>
        <v-col class="col-6 py-1">
          {{ vat.getPrice() | currency }}
        </v-col>
      </v-row>

      <!-- tax number -->
      <v-row v-if="showTaxNr">
        <v-col class="col-6 py-1">
          {{ $t('paymentTotalListing.taxNr') }}
        </v-col>
        <v-col class="col-6 py-1">
          {{ booking.getDestination().getTaxNr() }}
        </v-col>
      </v-row>
    </v-col>

    <!-- total -->
    <v-col
      v-if="booking.getCancelledBookingEntries().length"
      class="col-12 py-0 border-bottom font-weight-black"
    >
      <v-row>
        <v-col class="col-6">
          <!-- booking has cancellations -->
          <span>
            {{ $t('adminBasket.totalInclVat') }}
          </span>
        </v-col>
        <v-col class="col-6">
          {{ booking.getTotal() | currency }}
        </v-col>
      </v-row>
    </v-col>

    <!-- cancellations if any -->
    <v-col
      v-if="booking.getCancelledBookingEntries().length"
      class="col-12 py-0 border-bottom"
    >
      <v-row>
        <v-col class="col-6">
          {{ $t('paymentTotalListing.cancelledServices') }}
        </v-col>
        <v-col class="col-6">
          - {{ booking.getCancelledEntriesAmount() | currency }}
        </v-col>
      </v-row>
    </v-col>

    <!-- total without cancellations amount -->
    <v-col class="col-12 py-0 border-bottom font-weight-black">
      <v-row>
        <v-col class="col-6">
          <!-- paid -->
          <span v-if="booking.isPaid()">
            <span class="fal fa-check green--text mr-2" />
            {{ $t('shopBackendBookingDetail.paid') }}
          </span>

          <!-- unpaid -->
          <span v-else class="red--text">
            <span class="fal fa-times mr-2 red--text" />
            {{ $t('shopBackendBookingDetail.notPaid') }}
          </span>
        </v-col>
        <v-col class="col-6">
          <span :class="booking.isPaid() ? 'green--text' : 'red--text'">
            {{ booking.getTotalWithoutCancelledBookingEntries() | currency }}
          </span>
        </v-col>
      </v-row>
    </v-col>
  </v-row>
</template>

<script>
import Booking from '@/classes-shared/bookings/Booking'
import cwc from '@/assets/icons/cwc.svg'

export default {
  name: 'PaymentTotalListing',
  props: {
    booking: {
      type: Booking,
      required: true,
    },
    showTaxNr: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  data() {
    return {
      cwc,
    }
  },
}
</script>
