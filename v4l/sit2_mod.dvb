/*+++ *******************************************************************\ 
*     SIT2  - DVB-T2/T/C demodulator and tuner
*
*     --------------------------------------------------------------- 
*     This software is provided "AS IS" without warranty of any kind, 
*     either expressed or implied, including but not limited to the 
*     implied warranties of noninfringement, merchantability and/or 
*     fitness for a particular purpose.
*     --------------------------------------------------------------- 
*   
*     Copyright (c) 2013 ShenZhen Bestunar Ltd,Inc.
*     All rights reserved. 
*     Max Nibble <nibble.max@gmail.com>
*
*
\******************************************************************* ---*/ 
#include <linux/delay.h>
#include <linux/errno.h>
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/string.h>
#include <asm/div64.h>
#include "dvb_frontend.h"
#include "sit2.h"
#include "sit2_op.h"

static void sit2_mod_release(struct dvb_frontend *fe)
{
	struct sit2_mod_state *m_state = fe->demodulator_priv;
	kfree(m_state);
}

static int sit2_mod_init(struct dvb_frontend *fe)
{
	struct sit2_mod_state *m_state = fe->demodulator_priv;
	struct sit2_state *state = &m_state->drv_state;
	return sit2_drv_init(state);
}

static int sit2_mod_sleep(struct dvb_frontend *fe)
{
	struct sit2_mod_state *m_state = fe->demodulator_priv;
	struct sit2_state *state = &m_state->drv_state;
	return sit2_drv_sleep(state);
}

static int sit2_mod_get_frontend_algo(struct dvb_frontend *fe)
{
	return DVBFE_ALGO_HW;
}

static int sit2_mod_set_frontend(struct dvb_frontend *fe)
{
	struct dtv_frontend_properties *c = &fe->dtv_property_cache;
	struct sit2_mod_state *m_state = fe->demodulator_priv;
	struct sit2_state *state = &m_state->drv_state;
	
	return sit2_drv_set_frontend(state, c->delivery_system, c->frequency, c->bandwidth_hz,
	c->symbol_rate, c->modulation, c->stream_id);	
}

static int sit2_mod_read_status(struct dvb_frontend *fe, fe_status_t *status)
{
	struct sit2_mod_state *m_state = fe->demodulator_priv;
	struct sit2_state *state = &m_state->drv_state;
	return sit2_drv_read_status(state, status);
}

static int sit2_mod_tune(struct dvb_frontend *fe,
			bool re_tune,
			unsigned int mode_flags,
			unsigned int *delay,
			fe_status_t *status)
{	
	*delay = HZ / 5;	
	if (re_tune) {
		int ret = sit2_mod_set_frontend(fe);
		if (ret)
			return ret;
	}	
	return sit2_mod_read_status(fe, status);
}

static int sit2_mod_get_frontend(struct dvb_frontend *fe)
{
	struct dtv_frontend_properties *c = &fe->dtv_property_cache;
	struct sit2_mod_state *m_state = fe->demodulator_priv;
	struct sit2_state *state = &m_state->drv_state;

	return sit2_drv_get_frontend(state, &c->symbol_rate, &c->modulation, &c->transmission_mode,
	&c->guard_interval, &c->hierarchy, &c->code_rate_HP, &c->code_rate_LP, &c->inversion, &c->fec_inner);
}

static int sit2_mod_read_snr(struct dvb_frontend *fe, u16 *snr)
{
	struct sit2_mod_state *m_state = fe->demodulator_priv;
	struct sit2_state *state = &m_state->drv_state;
	return sit2_drv_read_snr(state, snr);
}

static int sit2_mod_read_ber(struct dvb_frontend *fe, u32 *ber)
{
	struct sit2_mod_state *m_state = fe->demodulator_priv;
	struct sit2_state *state = &m_state->drv_state;
	return sit2_drv_read_ber(state, ber);	
}

static int sit2_mod_read_ucblocks(struct dvb_frontend *fe, u32 *ucblocks)
{
	struct sit2_mod_state *m_state = fe->demodulator_priv;
	struct sit2_state *state = &m_state->drv_state;
	return sit2_drv_read_ucblocks(state, ucblocks);
}

static int sit2_mod_read_signal_strength(struct dvb_frontend *fe, u16 *strength)
{
	struct sit2_mod_state *m_state = fe->demodulator_priv;
	struct sit2_state *state = &m_state->drv_state;
	return sit2_drv_read_signal_strength(state, strength);
}

static const struct dvb_frontend_ops sit2_ops = {

	/* DVB-T, DVB-T2 and DVB-C */
	.delsys = { SYS_DVBT, SYS_DVBT2, SYS_DVBC_ANNEX_A },

	/* DVB-C only */
	/*.delsys = { SYS_DVBC_ANNEX_A },*/

	.info = {
		.name = "Sit2 DVB-T2/C",
		.frequency_stepsize = 62500,
		.frequency_min = 48000000,
		.frequency_max = 870000000,
		.symbol_rate_min = 870000,
		.symbol_rate_max = 7500000,
		.caps =	FE_CAN_FEC_1_2			|
			FE_CAN_FEC_2_3			|
			FE_CAN_FEC_3_4			|
			FE_CAN_FEC_5_6			|
			FE_CAN_FEC_7_8			|
			FE_CAN_FEC_AUTO			|
			FE_CAN_QPSK			|
			FE_CAN_QAM_16			|
			FE_CAN_QAM_32			|
			FE_CAN_QAM_64			|
			FE_CAN_QAM_128			|
			FE_CAN_QAM_256			|
			FE_CAN_QAM_AUTO			|
			FE_CAN_TRANSMISSION_MODE_AUTO	|
			FE_CAN_GUARD_INTERVAL_AUTO	|
			FE_CAN_HIERARCHY_AUTO		|
			FE_CAN_MUTE_TS			|
			FE_CAN_2G_MODULATION		|
			FE_CAN_MULTISTREAM
		},

	.release		= sit2_mod_release,
	.init			= sit2_mod_init,
	.sleep			= sit2_mod_sleep,

	.tune			= sit2_mod_tune,
	.set_frontend		= sit2_mod_set_frontend,
	.get_frontend		= sit2_mod_get_frontend,
	.get_frontend_algo	= sit2_mod_get_frontend_algo,

	.read_status		= sit2_mod_read_status,
	.read_snr		= sit2_mod_read_snr,
	.read_ber		= sit2_mod_read_ber,
	.read_ucblocks		= sit2_mod_read_ucblocks,
	.read_signal_strength	= sit2_mod_read_signal_strength,
};

struct dvb_frontend *sit2_attach(const struct sit2_config *config,
		struct i2c_adapter *i2c)
{
	struct sit2_mod_state *m_state = NULL;
	m_state = kzalloc(sizeof(struct sit2_mod_state), GFP_KERNEL);
	
	if (!m_state) {
#ifdef _SIT2_DEBUG_
		dev_err(&i2c->dev, "%s: kzalloc() failed\n",
				KBUILD_MODNAME);
#endif
		goto error;
	}	
	sit2_op_attach(&m_state->drv_state, i2c, config->ts_bus_mode, config->ts_clock_mode);
	
	memcpy(&m_state->frontend.ops, &sit2_ops,
	       sizeof(struct dvb_frontend_ops));
	m_state->frontend.demodulator_priv = m_state;
	return &m_state->frontend;
error:
	kfree(m_state);
	return NULL;
}
EXPORT_SYMBOL(sit2_attach);

MODULE_DESCRIPTION("sit2 demodulator driver");
MODULE_AUTHOR("Max Nibble <nibble.max@gmail.com>");
MODULE_LICENSE("GPL");
